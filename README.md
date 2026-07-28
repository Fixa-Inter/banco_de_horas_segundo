# FIXA - Banco de Horas

Aplicação web para acompanhamento de horas semanais da equipe do segundo ano do **Fixa**, projeto integrado diretamente ao GitHub Projects.

---

## Motivação

Em projetos acadêmicos com múltiplos repositórios e membros trabalhando, é difícil ter visibilidade real de quanto cada pessoa está contribuindo semana a semana. Planilhas manuais não funcionam, e ferramentas externa adicionam fricção desnecessária ao fluxo de trabalho, o que dificultaria nosso trabalho ao termos que adicionar as horas, comentários, detalhes de tudo que fizemos em outra plataforma.

O Projeto do Banco de Horas resolve isso de forma simples, onde usa **o próprio GitHub como fonte de dados**, sem banco de dados externo, sem backend, e sem que a equipe precise sair do ambiente que já utiliza no dia a dia.

---

##  Objetivo

Permitir que a equipe acompanhe o cumprimento da **meta mínima de 3 horas semanais** por membro (definido entre o próprio grupo), com base nas horas registradas diretamente nas issues do GitHub Projects, o que qualquer pessoa, de qualquer máquina, pode ver os mesmos dados ao abrir o site.

---

## Como funciona

### Registro de horas

As horas são registradas em dois lugares no GitHub Projects:

**Campo numérico `Horas Trabalhadas`**: preenchido na issue com o total acumulado de horas dedicadas àquela tarefa. Esse valor é usado nas métricas gerais do projeto (total por membro, por sprint, por matéria).

**Comentários com `+Xh`**: ao atualizar o campo, cada integrante adiciona um comentário na issue indicando o tempo dedidado daquela semana. É esse comentário que o sistema usa para calcular o banco de horas semanal.

Exemplo de fluxo:

```
Issue: "Modelagem Conceitual do Banco de Dados"
Assignee: alineteodoro

Semana 1 (20/jul):
  → comenta: "+2h levantamento de requisitos"
  → atualiza campo: 0 → 10 (minutos)

Semana 2 (27/jul):
  → comenta: "+3h diagrama ER e revisão"
  → atualiza campo: 120  → 300 (minutos)
```

O sistema contabiliza cada semana de forma independente, mesmo que seja a mesma issue, ou seja, nós conseguimos resolver o problema do cumprimento da meta semanal sem depender de ferramentas externas.

--- 

### Formatos aceitos nos comentários

O comentário precisa começar com os valores no seguinte formato:

| Formato | Equivalente |
|---|---|
| `+30m` | 30 minutos |
| `+2h30m` | 2 horas e 30 minutos |

O restante do comentário pode ser qualquer texto, já que o sistema só lê o início da linha.

### Campo numérico `Horas Trabalhadas`

O campo deve ser preenchido sempre em minutos:

| Valor | Interpretação |
|---|---|
| `30` | 30 minutos |
| `150` | 2 horas e 30 minutos |

Os formatos como 2h ou 30m são exclusivos dos comentários, então sempre preencha o campo em minutos.

## Métricas disponíveis
### Visão geral
<img width="1858" height="880" alt="image" src="https://github.com/user-attachments/assets/56aaf8b3-1a5d-4a30-af37-07224294f12d" />
- Issues totais do 2° ano
- Total de horas trabalhadas pelo grupo
- Total de tasks criadas
- Total de reaberturas de tasks
- Taxa de conclusão do projeto: com base nas tasks em Backlog, Em espera, Em progresso e Em revisão em relação às Finalizadas
- Taxa de retrabalho — proporção de tasks que foram reabertas 

### Por Integrante
<img width="1858" height="882" alt="image" src="https://github.com/user-attachments/assets/72f64a5b-f3cf-43fc-9329-be3494605006" />
- Horas trabalhadas na semana atual com indicador de meta cumprida ou não
- Total de horas trabalhadas desde o início
- Tasks alocadas atualmente
- Quantidade de code reviews realizados
- Quantidade de reaberturas de tasks
- Comentários feitos nas tasks para análise mais detalhada da contribuição e justificava das horas trabalhadas

### Do Grupo
<img width="1855" height="886" alt="image" src="https://github.com/user-attachments/assets/849ff349-9fc1-4531-af70-cf5fb2dcaa20" />
- Total de issues criadas
- Quantidade de tasks finalizadas e em andamento
- Quantidade de reaberturas de issues
- Por sprint: total de issues, abertas e finalizadas
- Rank das tasks que mais demoram para ser realizadas (com base nos campos Data de Início e Data de Término)
- Taxa de conclusão do projeto e taxa de retrabalho
- Quantidade total de issues em cada estado (Backlog, Em progresso, Em revisão, Finalizado)

---

## Autenticação

A aplicação usa **Personal Access Token (PAT)** do GitHub.

### Como gerar seu token (feito uma vez por membro)
<img width="1790" height="410" alt="image" src="https://github.com/user-attachments/assets/7f721104-8845-4341-9d47-f036652cf0d9" />
1. Acesse github.com → foto de perfil → **Settings**
2. **Developer settings** → **Personal access tokens** → **Fine-grained tokens**
3. **Generate new token**
4. Em **Resource owner**, selecione a organização `Fixai-Inter`
5. Em **Repository access**, selecione **All repositories**
6. Configure as permissões:
    **Repository permissions:** `Issues` → Read-only
    **Organization permissions:** `Projects` → Read-only, `Members` → Read-only
    <img width="1030" height="523" alt="image" src="https://github.com/user-attachments/assets/e7fda423-4aba-43d4-b2e5-c8df3ad3ffe7" />
    <img width="1022" height="524" alt="image" src="https://github.com/user-attachments/assets/920b78dd-fb1b-4c46-a6bd-ec487e031e0a" />
7. Gere e copie o token (`github_pat_...`)
8. Cole no campo de autenticação do site ao abrir

O token fica salvo na sessão do navegador e some ao fechar. Nenhuma informação sensível é armazenada no repositório ou enviada a servidores externos, mas o login de cada usuário fica guardado na máquina. Caso acesse por uma máquina que não seja a sua, não se esqueça de sair do site com o seu acesso pessoal.

---

##  Estrutura do projeto

```
src/
├── components/        # componentes de UI
├── services/
│   ├── GitHubDataService.ts    # query GraphQL + normalização dos dados
│   └── WeeklyHoursService.ts  # cálculo de horas semanais por comentários
├── auth/
│   └── GitHubAuth.tsx         # autenticação via PAT
└── App.tsx
```

## Campos do GitHub Projects utilizados

| Campo | Tipo | Uso |
|---|---|---|
| `Horas Trabalhadas` | Número | Total acumulado da issue, usado em métricas gerais |
| `Status` | Select | Situação atual da issue no kanban |
| `Sprint` | Iteration | Agrupamento por sprint(datas já bem definidas) |
| `Tipo` | Select | Diferencia Tasks de Épicos |
| `Ano` | Select | Filtra issues do 2º Ano |
| `Data de Início` | Data | Referência temporal da issue |
| `Data de Término` | Data | Referência de conclusão |

---

## Público-alvo
- **Integrantes da equipe**: acompanhamento do próprio progresso e da equipe semana a semana.
