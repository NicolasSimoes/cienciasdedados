# Desempenho de cursos no ENADE 2023 — Sprints 1, 2 e 3

Projeto de extensão da disciplina **Ciência de Dados (T326)** · UNIFOR · Profa. Dra. Cynthia Moreira Maia.
Equipe: Nicolas Simões Leão Santos, Gisele Costa Siqueira, Silvio Bruno do Nascimento Lima, Marcos Vinicius Herbster, Luiz Eduardo Alves.

## O que o projeto faz
Classificação supervisionada **no nível do curso**: dado o perfil médio dos concluintes de um curso e as características do curso
(categoria administrativa, organização acadêmica, modalidade, região, UF e área), o modelo prevê se a **nota média do curso no ENADE 2023
fica acima ou abaixo da mediana da sua área**. Os fatores mais associados aos cursos acima da mediana servem de base para um relatório
comparativo por curso (previsto para as próximas sprints).

> Os microdados do INEP são embaralhados por arquivo (LGPD); os 32 arquivos só podem ser unidos pelo código do curso (`CO_CURSO`).
> Por isso a unidade de análise é o **curso**, não o aluno.

## Conteúdo
| Arquivo | Descrição |
|---|---|
| `ENADE2023_Sprints_1_2_3.ipynb` | Notebook com as três sprints, já executado (Parte 1 = Sprint 1, Parte 2 = Sprint 2, Parte 3 = Sprint 3) |
| `requirements.txt` | Dependências |
| `figuras/` | Gráficos gerados pelo notebook |
| `dados_processados/cursos_enade2023_modelagem.csv` | Tabela final por curso (resultado do pré-processamento) |
| `resultados/` | Métricas de validação e teste, hiperparâmetros, ablação e coeficientes (CSV) |

## Como executar
1. Baixe os *Microdados do Enade 2023* em <https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos/microdados/enade> e salve `microdados_enade_2023.zip` ao lado do notebook (ou aponte a variável de ambiente `ENADE_ZIP`). Não precisa descompactar.
2. `pip install -r requirements.txt`
3. `jupyter notebook ENADE2023_Sprints_1_2_3.ipynb` e execute todas as células (alguns minutos).

Semente aleatória fixa (`RANDOM_STATE = 42`): os resultados se repetem.

## Resultados principais (Sprint 3)
7.723 cursos (≥ 10 concluintes válidos) · divisão agrupada por IES: 80% treino / 20% teste, validação cruzada de 5 partes dentro do treino.

| Algoritmo | Acurácia (validação, média ± dp) | Acurácia (teste) | AUC-ROC (teste) |
|---|---|---|---|
| Baseline (classe mais frequente) | 50,1 ± 0,0 | 50,1 | 0,500 |
| **Regressão Logística** | **80,4 ± 2,1** | **81,9** | 0,906 |
| SVM (RBF) | 80,4 ± 2,4 | 81,7 | 0,909 |
| Rede Neural (MLP) | 79,9 ± 1,8 | 81,7 | 0,902 |
| KNN | 75,5 ± 3,2 | 74,5 | 0,841 |
| Árvore de Decisão | 73,7 ± 3,1 | 75,1 | 0,827 |
| Naive Bayes | 73,8 ± 2,4 | 73,8 | 0,800 |

Regressão Logística, SVM e MLP empatam tecnicamente; os testes estatísticos entre modelos e os comitês de classificadores ficam para as próximas sprints.
