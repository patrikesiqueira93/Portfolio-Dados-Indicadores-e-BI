🛠️ Documentação Técnica & Arquitetura de Dados | Control Tower DE&BI💻 

Visão Geral Técnica:
Esta documentação detalha os aspectos de Engenharia de Dados, Modelagem Dimensional, Linguagem DAX, Especificações JSON/Vega-Lite e UI/UX Avançado aplicados no desenvolvimento do dashboard executivo Control Tower de Manutenção & SLA.
🤖 Engenharia de Prompts & Co-Criação com Inteligência ArtificialUm dos pilares tecnológicos e metodológicos deste projeto foi a aplicação estratégica de Engenharia de Prompts e Assistência por Inteligência Artificial (Gemini AI) para aceleração do ciclo de desenvolvimento de software e BI.
+-----------------------------------------------------------------------------------+
|                        FLUXO DE CO-CRIAÇÃO COM IA (GEMINI)                        |
+-------------------+--------------------+--------------------+---------------------+
| 1. CONCEITO UI    | 2. ASSETS SVG      | 3. DAX & HTML      | 4. DENEB / VEGA     |
| UX Layout 16:9    | Backgrounds &      | Medidas HTML com   | Gauge Analógico com |
| & Paleta Dark     | Watermarks Vector  | CSS Inline & Badges| Camadas & Trig.     |
+-------------------+--------------------+--------------------+---------------------+

Contexto de Origem & Incentivo Acadêmico:UNIVESP (4º Semestre de Engenharia da Computação): Projeto desenvolvido como aplicação prática das disciplinas de Banco de Dados, Engenharia de Software e Interface Homem-Computador (IHC).Educathon Eldorado + IBM: Metodologia de construção de prompts estruturados baseada nos aprendizados da parceria entre o Instituto de Pesquisas Eldorado e o curso de Prompt Engineering da IBM, focando em:Decomposição modular de problemas complexos.Refatoração iterativa de código.Validação rigorosa de sintaxe e performance.Google Gemini AI como Co-Piloto:Criação e Refinamento de Código: Geração de fórmulas DAX complexas com tratamento de aspas para renderização HTML/CSS em tempo de execução.Desenvolvimento Declarativo JSON: Cálculo e ajuste de camadas trigonométricas (theta, arc, radianos) para o Gauge Analógico em Deneb/Vega-Lite.Design Visual & UI Assets: Concepção da paleta de cores executiva (#0F172A, #2563EB, #10B981, #EF4444) e estrutura dos arquivos SVG de tela de fundo e tooltip.🏗️ 1. Arquitetura da Solução & Pipeline de DadosO pipeline de dados segue os princípios da Arquitetura Medallion, garantindo rastreabilidade, imutabilidade da fonte e alta performance no consumo analítico:[ Fontes Brutas / CSVs ] 

┌──────────────────────────┐
│   Camada BRONZE (Raw)    │ ──> Ingestão em arquivos Parquet preservando schema original.
└──────────────────────────┘
          │
          ▼
┌──────────────────────────┐
│  Camada SILVER (Cleansed)│ ──> Tratamento de nulos, tipagem rígida, cálculo de tempos em
└──────────────────────────┘     horas operacionais e marcação de flags de SLA.
          │
          ▼
┌──────────────────────────┐
│   Camada GOLD (Star)     │ ──> Modelagem dimensional pronta para consumo em alta velocidade.
└──────────────────────────┘
          │
          ▼
┌──────────────────────────┐
│  Power BI (Control Tower)│ ──> Camada de visualização executiva e inteligência de negócios.
└──────────────────────────┘

📐 2. Modelagem de Dados (Star Schema)A camada analítica foi estruturada no modelo Esquema Estrela (Star Schema) com relacionamentos $1:N$ unidirecionais e cardinalidade controlada:                   ┌───────────────────────┐
│     dim_tempo         │
└───────────────────────┘
          │ 1
          │
          │ N
┌───────────────────────┐  N ┌───────────────────────┐
│   dim_equipamento     │───>│    fato_chamados      │
└───────────────────────┘    └───────────────────────┘
           │
           │
           ▼
┌───────────────────────┐
│       _Medidas        │ (Tabela Técnica / Repositório DAX)
└───────────────────────┘

Detalhes do Dicionário de Dados:fato_chamados (Fato): Registra eventos de ordens de manutenção, contendo id_chamado, id_ativo, dt_abertura, dt_fechamento, tempo_atendimento_horas, custo_manutencao e fl_estourou_sla (0 ou 1).dim_equipamento (Dimensão): Atributos dos ativos (id_ativo, categoria_equipamento, filial_operacao, modelo, ano_fabricacao).dim_tempo (Dimensão): Calendário contínuo em PT-BR para análises temporais por Mês, Trimestre, Ano e Dia da Semana._Medidas (Repositório): Tabela exclusiva para centralização e organização das regras de negócio em DAX.📊 3. Principais Formulações DAXAs métricas do projeto combinam cálculos estatísticos operacionais, acúmulos físico-financeiros e formatação dinâmica em HTML.3.1. Métricas Principais de NegócioTotal de Chamados:Total Chamados = COUNTROWS(fato_chamados)

Custo Total de Manutenção (OPEX):Custo Manutencao = SUM(fato_chamados[custo_manutencao])
Mean Time to Repair (MTTR em Horas):MTTR Horas = AVERAGE(fato_chamados[tempo_atendimento_horas])
Percentual de Cumprimento de SLA:% Cumprimento SLA = 
VAR _Total = [Total Chamados]
VAR _Atrasos = CALCULATE(COUNTROWS(fato_chamados), fato_chamados[fl_estourou_sla] = 1)
RETURN
DIVIDE(_Total - _Atrasos, _Total, 0)
Custo Acumulado (Curva S):Custo_Acumulado = 
CALCULATE(
    [Custo Manutencao],
    FILTER(
        ALLSELECTED(dim_tempo),
        dim_tempo[dt_abertura_chamado] <= MAX(dim_tempo[dt_abertura_chamado])
    )
)
Ativos Críticos Fora da Meta:Ativos_Criticos = 
CALCULATE(
    DISTINCTCOUNT(dim_equipamento[id_ativo]),
    FILTER(
        dim_equipamento,
        [% Cumprimento SLA] < 0.80
    )
)
3.2. Medidas HTML/CSS para Cards CustomizadosPara renderização via suplemento HTML Content, construímos componentes visuais nativos com CSS Inline, evitando dependências externas:Card_HTML_AtivosCriticos = 
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
🎨 4. Visual Declarativo Deneb / Vega-Lite (Gauge Analógico)O indicador da Disponibilidade da Frota (%) foi construído do zero via especificação Vega-Lite v5 no visual Deneb, simulando um manômetro industrial avançado.Destaques da Implementação JSON:Varredura Angular de $240^\circ$: Conversão de percentuais em radianos (início em -2.0944 rad e término em 2.0944 rad).Gradiente Tricolor de Progresso: Escala dinâmica variando de Vermelho (#EF4444) $\rightarrow$ Amarelo (#F59E0B) $\rightarrow$ Verde (#10B981).Ponteiro de Precisão Trigonômico: Projeção via vetores sin/cos com controle de eixo ("axis": null) para eliminar grids indesejadas.{
  "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
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
    }
  ]
}
          
🔄 5. Fluxo de Versionamento & Governança no GitO desenvolvimento seguiu o fluxo de trabalho Git Feature Branch com padronização de commits (Conventional Commits):# 1. Rastreio de alterações locais
git status

# 2. Inclusão dos arquivos salvos no Staging Area
git add .

# 3. Commit semântico estruturado
git commit -m "feat(final): conclui dashboard executivo de manutencao e SLA com 4 paginas e tooltips customizados"

# 4. Sincronização com o repositório remoto no GitHub
git push origin main

🏆 Conclusão: A combinação de Engenharia de Dados estruturada em Star Schema, medidas analíticas em DAX, componentes HTML/CSS e especificação gráfica avançada em Deneb resultou em uma solução completa de nível industrial, demonstrando autonomia técnica e alinhamento com as melhores práticas do mercado corporativo.
