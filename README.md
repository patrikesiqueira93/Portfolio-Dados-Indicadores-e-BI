# 🛠️ Documentação Técnica & Arquitetura de Dados | Control Tower DE&BI

Visão Geral Técnica: Esta documentação detalha os aspectos de Engenharia de Dados, Modelagem Dimensional, Linguagem DAX, Especificações JSON/Vega-Lite e UI/UX Avançado aplicados no desenvolvimento do dashboard executivo Control Tower de Manutenção & SLA.

---

## 🤖 Engenharia de Prompts & Co-Criação com Inteligência Artificial

Um dos pilares tecnológicos e metodológicos deste projeto foi a aplicação estratégica de Engenharia de Prompts e Assistência por Inteligência Artificial (**Google Gemini AI**) para aceleração do ciclo de desenvolvimento de software e BI.

```text
+-------------------------------------------------------------------------------+
|                        FLUXO DE CO-CRIAÇÃO COM IA (GEMINI)                    |
+-------------------+-------------------+-------------------+-------------------+
| 1. CONCEITO UI    | 2. ASSETS SVG     | 3. DAX & HTML     | 4. DENEB / VEGA   |
| UX Layout 16:9    | Backgrounds &     | Medidas HTML com  | Gauge Analógico   |
| Paleta Dark       | Watermarks Vector | CSS Inline &      | Camadas & Trig.   |
| Slate & Blue      | para Tooltip      | Badges Dinâmicos  | para Ponteiro     |
+-------------------+-------------------+-------------------+-------------------+

Contexto de Origem & Incentivo Acadêmico:
UNIVESP (4º Semestre de Engenharia da Computação): Projeto desenvolvido como aplicação prática das disciplinas de Banco de Dados, Engenharia de Software e Interface Homem-Computador (IHC).

Educathon Eldorado + IBM: Metodologia de construção de prompts estruturados baseada nos aprendizados da parceria entre o Instituto de Pesquisas Eldorado e o curso de Prompt Engineering da IBM, focando em:

Decomposição modular de problemas complexos.

Refatoração iterativa de código.

Validação rigorosa de sintaxe e performance.

Google Gemini AI como Co-Piloto Tecnológico:
Criação e Refinamento de Código: Geração de fórmulas DAX complexas com tratamento rigoroso de aspas para renderização HTML/CSS em tempo de execução no Power BI.

Desenvolvimento Declarativo JSON: Cálculo e ajuste de camadas trigonométricas (theta, arc, radianos) para o Gauge Analógico em Deneb/Vega-Lite.

Design Visual & UI Assets: Concepção da paleta de cores executiva (#0F172A, #2563EB, #10B981, #EF4444) e estruturação visual dos arquivos SVG de tela de fundo e tooltip flutuante.

🏗️ 1. Arquitetura da Solução & Pipeline de Dados
O pipeline de dados segue os princípios da Arquitetura Medallion, garantindo rastreabilidade, imutabilidade da fonte e alta performance no consumo analítico:

[ Fontes Brutas / CSVs ]
          │
          ▼
┌───────────────────────────┐
│   Camada BRONZE (Raw)     │ ──► Ingestão preservando schema original.
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│   Camada SILVER (Cleaned) │ ──► Tratamento de nulos, tipagem rígida,
└─────────────┬─────────────┘     cálculo de tempos e flags de SLA.
              │
              ▼
┌───────────────────────────┐
│   Camada GOLD (Star)      │ ──► Modelagem dimensional otimizada
└─────────────┬─────────────┘     pronta para consumo analítico.
              │
              ▼
┌───────────────────────────┐
│ Power BI (Control Tower)  │
└───────────────────────────┘

📐 2. Modelagem Dimensional (Esquema Estrela / Star Schema)
A modelagem de dados foi estruturada rigorosamente no padrão Star Schema (Fato e Dimensões):

fato_manutencao: Tabela Fato contendo os registros de ordens de serviço (OS), datas de abertura e fechamento, custos e status.

dim_equipamento: Tabela Dimensão contendo a hierarquia de ativos (id_ativo, categoria_equipamento, filial_operacao).

dim_tempo: Tabela Dimensão Calendário para inteligência temporal contínua (dt_abertura_chamado, Ano, Mês/Ano).

┌─────────────────────────┐
│     dim_equipamento     │
├─────────────────────────┤
│ id_ativo (PK)           │
│ categoria_equipamento   │
│ filial_operacao         │
└────────────┬────────────┘
             │ 1
             │
             │ N
┌────────────┴────────────┐         ┌─────────────────────────┐
│     fato_manutencao     │ N     1 │        dim_tempo        │
├─────────────────────────┼─────────┤─────────────────────────┤
│ id_chamado (PK)         │         │ dt_abertura_chamado(PK) │
│ id_ativo (FK)           │         │ Ano                     │
│ dt_abertura (FK)        │         │ Mes_Nome                │
│ tempo_atendimento_horas │         │ Mes_Ano                 │
│ custo_manutencao        │         └─────────────────────────┘
└─────────────────────────┘

🧮 3. Engenharia de Métricas DAX & Renderização HTML
Abaixo estão os códigos DAX centrais da aplicação, cobrindo calculabilidade temporal, acumulados e visuais dinâmicos em HTML:

A. Acumulado Físico-Financeiro (Curva S)

Custo_Acumulado = 
CALCULATE(
    [Custo Manutencao],
    FILTER(
        ALLSELECTED(dim_tempo),
        dim_tempo[dt_abertura_chamado] <= MAX(dim_tempo[dt_abertura_chamado])
    )
)

B. Contagem de Ativos Críticos (SLA < 80%)

Ativos_Criticos = 
CALCULATE(
    DISTINCTCOUNT(dim_equipamento[id_ativo]),
    FILTER(
        dim_equipamento,
        [% Cumprimento SLA] < 0.80
    )
)

C. Card HTML Padronizado para Sidebar (Card_HTML_AtivosCriticos)

Card_HTML_AtivosCriticos = 
VAR ValorCriticos = FORMAT([Ativos_Criticos], "#,##0") & " equip."
VAR CorBadge = "#EF4444"
VAR FundoBadge = "#EF44441A"

RETURN
"
<div style='
    font-family: Segoe UI, sans-serif;
    background-color: #FFFFFF;
    border-radius: 10px;
    padding: 16px;
    box-shadow: 0px 4px 12px rgba(0, 0, 0, 0.04);
    border-left: 5px solid " & CorBadge & ";
'>
    <div style='font-size: 11px; color: #64748B; font-weight: 600; text-transform: uppercase; letter-spacing: 0.5px;'>
        Ativos Fora da Meta SLA
    </div>
    <div style='font-size: 28px; font-weight: 700; color: #0F172A; margin: 6px 0;'>
        " & ValorCriticos & "
    </div>
    <div style='display: inline-block; font-size: 10px; font-weight: 700; color: " & CorBadge & "; background-color: " & FundoBadge & "; padding: 3px 8px; border-radius: 4px;'>
        SLA CRÍTICO (< 80%)
    </div>
</div>
"

🎨 4. Especificação Declarativa do Gauge Analógico em Deneb (Vega-Lite)
Código JSON completo utilizado no suplemento Deneb para construção do manômetro analógico com gradiente tricolor dinâmico e agulha angular:

{
  "$schema": "[https://vega.github.io/schema/vega-lite/v5.json](https://vega.github.io/schema/vega-lite/v5.json)",
  "data": {"name": "dataset"},
  "layer": [
    {
      "name": "ARCO_DE_FUNDO",
      "mark": {
        "type": "arc",
        "innerRadius": 90,
        "outerRadius": 155,
        "startAngle": -2.0944,
        "endAngle": 2.0944,
        "color": "#E2E8F0",
        "cornerRadius": 1
      }
    },
    {
      "name": "ARCO_DE_PROGRESSO_GRADIENTE",
      "transform": [
        {
          "calculate": "-2.0944 + (datum['% Cumprimento SLA'] * 4.1888)",
          "as": "angulo_final"
        }
      ],
      "mark": {
        "type": "arc",
        "innerRadius": 90,
        "outerRadius": 155,
        "startAngle": -2.0944,
        "cornerRadius": 6,
        "color": {
          "gradient": "linear",
          "stops": [
            {"offset": 0, "color": "#EF4444"},
            {"offset": 0.45, "color": "#F59E0B"},
            {"offset": 1, "color": "#10B981"}
          ]
        }
      },
      "encoding": {
        "theta2": {"field": "angulo_final", "type": "quantitative"}
      }
    },
    {
      "name": "MARCADOR_DE_META_80PCT",
      "mark": {
        "type": "arc",
        "innerRadius": 84,
        "outerRadius": 161,
        "startAngle": 1.2566,
        "endAngle": 1.2850,
        "color": "#0F172A"
      }
    },
    {
      "name": "PONTEIRO_AGULHA",
      "transform": [
        {
          "calculate": "-2.0944 + (datum['% Cumprimento SLA'] * 4.1888)",
          "as": "ang"
        },
        {
          "calculate": "87 * cos(datum['ang'] - 1.5708)",
          "as": "x2"
        }
      ],
      "mark": {
        "type": "rule",
        "stroke": "#0F172A",
        "strokeWidth": 4,
        "strokeCap": "round"
      },
      "encoding": {
        "x": {
          "field": "x2",
          "type": "quantitative",
          "scale": {"domain": [-165, 165]},
          "axis": null
        },
        "y": {
          "field": "y2",
          "type": "quantitative",
          "scale": {"domain": [-165, 165]},
          "axis": null
        },
        "x2": {"field": "x1"},
        "y2": {"field": "y1"}
      }
    },
    {
      "name": "NUCLEO_CENTRAL_ESCURO",
      "mark": {
        "type": "arc",
        "innerRadius": 0,
        "outerRadius": 68,
        "color": "#0F172A"
      }
    },
    {
      "name": "TEXTO_PERCENTUAL_CENTRAL",
      "mark": {
        "type": "text",
        "font": "Segoe UI",
        "fontSize": 22,
        "fontWeight": "bold",
        "color": "#F8FAFC",
        "dy": 0
      },
      "encoding": {
        "text": {
          "field": "% Cumprimento SLA",
          "type": "quantitative",
          "format": ".1%"
        }
      }
    },
    {
      "name": "ROTULO_DE_META",
      "mark": {
        "type": "text",
        "text": "META: 80.0%",
        "font": "Segoe UI",
        "fontSize": 18,
        "fontWeight": "bold",
        "color": "#64748B",
        "dy": 105
      }
    }
  ]
}

🛠️ 5. Versionamento e Governança de Código com Git / GitHub
A governança do projeto segue o fluxo de trabalho estruturado por commits semânticos (Conventional Commits):

feat: Implementação de novas funcionalidades (telas, medidas DAX, visuais).

style: Ajustes de UI/UX, bordas, cores e tipografia.

fix: Correção de erros de sintaxe e aspas.

Comandos de Sincronização Final:

git add .
git commit -m "docs: atualiza documentacao tecnica com fluxo de IA, star schema e json do deneb"
git push origin main

---

<Steps>
  <Step subtitle="Passo 1" title="Substituir o Conteúdo no VS Code">
    1. Abra o arquivo **`README.md`** no seu VS Code.
    2. Selecione todo o conteúdo existente (`Ctrl + A`) e apague.
    3. Cole todo o bloco de código acima.
    4. Salve o arquivo (`Ctrl + S`).
  </Step>

  <Step subtitle="Passo 2" title="Commit e Push no GitHub">
    Abra o terminal do VS Code e envie a documentação atualizada e perfeitamente formatada:
    
    ```bash
    git add .
    ```
    ```bash
    git commit -m "docs: corrige formatacao de marcacao e blocos de texto no README"
    ```
    ```bash
    git push origin main
    ```
  </Step>
</Steps>

---

## 🏁 Conclusão & Impacto Gerencial

A construção da **Control Tower de Manutenção & SLA** comprova a viabilidade e o alto valor agregado de integrar **Engenharia de Dados**, **Modelagem Dimensional em Star Schema**, **Design de Interface Corporativo (UI/UX)** e **Inteligência Artificial Generativa** no ciclo de vida de soluções analíticas.

### Principais Legados do Projeto:

1. **Tomada de Decisão Baseada em Dados (Data-Driven):** Redução drástica do tempo de identificação de gargalos de atendimento e desvios operacionais através da centralização de KPIs em 4 visuais altamente focados.
2. **Prevenção de Estouro Orçamentário:** Adoção da **Curva S Físico-Financeira**, permitindo que gestores identifiquem a taxa de consumo do OPEX de manutenção mês a mês de forma preditiva.
3. **Visibilidade em Nível de Ativo:** Integração da **Página Oculta de Tooltip**, fornecendo uma radiografia instantânea de qualquer equipamento crítico sem poluir o canvas principal ou exigir múltiplos cliques do executivo.
4. **Governança & Reprodutibilidade:** Versionamento rigoroso em Git/GitHub, garantindo rastreabilidade histórica de cada melhoria de código, fórmula DAX e especificação JSON desenvolvida.

---

## 👨‍💻 Autor & Contexto Profissional

**Patrik Ernandes Siqueira**  
*Estudante de Engenharia da Computação (UNIVESP) & Analista de Dados / Suporte Operacional em análise de dados na Petrobras (Bacia de Campos).*

* **LinkedIn:** [Acessar Perfil Professional]([https://linkedin.com](https://www.linkedin.com/in/patrik-e-siqueira/))
* **GitHub:** [Acessar Repositório do Projeto](https://github.com/patrikesiqueira93)
* **Email:** [Contato Profissional](mailto:patrikesiqueira@gmail.com)

---

> 💡 *Projeto desenvolvido como estudo prático avançado e portfólio de engenharia analytics, aplicando as melhores práticas de mercado e co-criação com inteligência artificial.*
