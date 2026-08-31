# Aplicando os perfis de stack

Trabalho no fork de `mattpocock/skills`, branch `feat/multi-stack-profiles`.

## 1. Branch e upstream

```bash
cd /Users/willianrattis/development/git/willianrattis/skills
git remote add upstream https://github.com/mattpocock/skills.git
git fetch upstream
git checkout -b feat/multi-stack-profiles
```

O `upstream` importa: sem ele, cada atualização do Matt vira cópia manual em
vez de merge.

## 2. Copiar os arquivos novos

Todos são adições. Nenhum sobrescreve arquivo do upstream.

```
stacks/dotnet/STACK.md
stacks/python/STACK.md
stacks/typescript/STACK.md
skills/engineering/use-stack/SKILL.md
.agents/stack            # opcional, ver abaixo
```

## 3. Ponteiro nas skills acopladas

Adicionar **um parágrafo**, logo após o `# Título` e antes da primeira seção,
nestes cinco arquivos:

- `skills/engineering/tdd/SKILL.md`
- `skills/engineering/codebase-design/SKILL.md`
- `skills/engineering/code-review/SKILL.md`
- `skills/engineering/implement/SKILL.md`
- `skills/engineering/prototype/SKILL.md`

Texto, idêntico nos cinco:

```md
Before applying anything below, load the active stack profile at
`stacks/<stack>/STACK.md`, where `<stack>` is the value in `.agents/stack`. If
that file is absent or empty, ask the user which stack this work targets and
write the answer there before continuing. This skill supplies the discipline;
the profile supplies the toolchain.
```

Uma linha, em posição previsível, em cinco arquivos. É o diff mínimo que
sobrevive a merge com o upstream.

## 4. Pergunta no início da sessão

No `CLAUDE.md` e no `AGENTS.md` da raiz do repo **onde as skills forem usadas**
(não neste repo de skills), adicionar:

```md
## Stack

This repo's skills are stack-aware. Before the first piece of code work in a
session, read `.agents/stack`. If it is missing or empty, ask which stack this
work targets and write the answer there. Do not infer it silently from file
extensions.
```

## 5. Um aviso sobre a opção escolhida

Você pediu resolução por pergunta no início da sessão. Funciona, mas com uma
ressalva que vale registrar aqui: a resposta vive no contexto, e o contexto é
compactado. Numa sessão longa o agente esquece o que você respondeu e volta a
adivinhar — normalmente adivinhando TypeScript, porque é o que os exemplos das
skills sugerem.

Por isso o desenho acima **persiste a resposta em `.agents/stack`**. A pergunta
continua acontecendo, uma vez, na primeira sessão do repo; da segunda em diante
o arquivo responde e o `/use-stack` existe pra trocar quando você quiser. Se
preferir a pergunta literalmente toda sessão, remova o passo de escrita do
`/use-stack` — mas aí conte com o esquecimento no meio das sessões longas.

## 6. Verificar

Não há teste automatizado pra isso. A verificação é operacional:

1. `echo dotnet > .agents/stack` num repo .NET seu
2. Peça uma feature pequena com `/tdd`
3. O agente deve propor `dotnet test --filter`, xUnit e um seam de endpoint,
   sem você ter mencionado nada disso

Se ele sugerir Vitest, o ponteiro do passo 3 não foi lido — provavelmente está
abaixo demais no arquivo.

## Commit

```bash
git add stacks skills/engineering/use-stack APPLY.md
git commit -m "feat: multi-stack profiles for stack-coupled skills"
```
