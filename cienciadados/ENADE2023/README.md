# Desempenho de cursos no ENADE 2023 — Sprints 1, 2, 3 e 4

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
| `ENADE2023_Sprints_1_2_3.ipynb` | Notebook com as quatro sprints, já executado (Parte 1 = Sprint 1, Parte 2 = Sprint 2, Parte 3 = Sprint 3, Parte 4 = Sprint 4: comitês de classificadores). Antes da Parte 1 há uma seção com a **estrutura da tabela de dados** (dimensões, colunas, tipos e exemplos) |
| `requirements.txt` | Dependências |
| `figuras/` | Gráficos gerados pelo notebook |
| `dados_processados/cursos_enade2023_modelagem.csv` | Tabela final por curso (resultado do pré-processamento) |
| `resultados/` | Métricas de validação e teste, hiperparâmetros, ablação e coeficientes (CSV); `comites_*.csv` e `comparacao_geral.csv` são da Sprint 4 |

## Como executar
1. Baixe os *Microdados do Enade 2023* em <https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos/microdados/enade> e salve `microdados_enade_2023.zip` ao lado do notebook (ou aponte a variável de ambiente `ENADE_ZIP`). Não precisa descompactar.
2. `pip install -r requirements.txt`
3. `jupyter notebook ENADE2023_Sprints_1_2_3.ipynb` e execute todas as células (alguns minutos; a Parte 4 treina 7 comitês e é a mais demorada).

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

Regressão Logística, SVM e MLP empatam tecnicamente; os comitês de classificadores foram feitos na Sprint 4 (abaixo) e os testes estatísticos entre modelos ficam para trabalhos futuros.

## Resultados principais (Sprint 4 — comitês de classificadores)
Mesmo protocolo da Sprint 3 (mesmo pré-processamento, mesma divisão por IES e as mesmas 5 partes de validação). Acurácia em %; tempo = treino final com todo o conjunto de treino (computador desta execução).

| Grupo | Modelo | Acurácia (validação, média ± dp) | Acurácia (teste) | AUC-ROC (teste) | Tempo de treino |
|---|---|---|---|---|---|
| Individual | **Regressão Logística** (referência) | 80,4 ± 2,1 | 81,9 | 0,906 | 0,2 s |
| Individual | Árvore de Decisão | 73,7 ± 3,1 | 75,1 | 0,827 | 0,7 s |
| Homogêneo | Bagging (Árvores) | 78,6 ± 2,1 | 79,0 | 0,883 | 35 s |
| Homogêneo | Random Forest | 78,9 ± 2,9 | 81,9 | 0,893 | 6,5 s |
| Homogêneo | AdaBoost (Árvores rasas) | 80,0 ± 2,0 | 79,5 | 0,896 | 17,5 s |
| Heterogêneo | Votação majoritária (3 melhores) | 80,5 ± 2,1 | 82,0 | — | 2,3 s |
| Heterogêneo | Votação por probabilidades (3 melhores) | 80,6 ± 2,3 | 82,3 | 0,909 | 7,3 s |
| Heterogêneo | Votação por probabilidades (6 algoritmos) | 78,9 ± 2,9 | 80,7 | 0,897 | 8,1 s |
| Heterogêneo | Empilhamento (6 algoritmos) | 80,8 ± 2,3 | 82,2 | 0,910 | 16,7 s |

- **Comitês homogêneos:** melhoram muito a Árvore de Decisão (+5 a +6 p.p., em 5 de 5 rodadas), mas só chegam perto da Regressão Logística.
- **Comitês heterogêneos:** com os 3 melhores algoritmos o ganho é de 0,1 a 0,4 p.p., menor que o erro de medida (≈ 2 p.p. entre rodadas; ≈ 1 p.p. no teste). Os 3 melhores concordam em 91% dos cursos, então votar quase não muda o resultado. Juntar os algoritmos mais fracos piora a votação.
- **Conclusão:** treze modelos chegam ao mesmo teto de ≈ 80–82% de acurácia, o que sugere que o limite está nos dados (perfil médio de poucos alunos por curso) e não no algoritmo. A Regressão Logística segue como modelo de referência: desempenho equivalente, treino mais rápido e coeficientes interpretáveis.
