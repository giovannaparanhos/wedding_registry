# Handoff — Site de Casamento Giovanna & Caio

## Contexto geral

Site estático de casamento hospedado em **GitHub Pages** com domínio customizado.  
Repositório: `giovannaparanhos/wedding_registry`  
URL: `https://giovannaparanhos.dev.br`  
Branch de trabalho atual: `claude/infallible-proskuriakova-785abc`  
Worktree local: `/Users/giovannaparanhos/Documents/casamento/wedding_registry/.claude/worktrees/infallible-proskuriakova-785abc`

> ⚠️ **Sempre trabalhe no worktree acima, não no diretório raiz do repo.**  
> Commits e pushes devem ir para o branch `claude/infallible-proskuriakova-785abc`.

---

## Arquivos do projeto

| Arquivo | Descrição |
|---|---|
| `index.html` | Página principal do casamento (09/07/2026) |
| `presentes.html` | Lista de presentes + PIX |
| `amigos.html` | Página da festa de amigos (11/07/2026) |
| `evento.ics` | Arquivo iCal para o casamento |
| `amigos.ics` | Arquivo iCal para a festa de amigos (existe no repo principal, não no worktree) |
| `.github/workflows/deploy.yml` | Workflow de deploy com injeção de secrets |

---

## Design

**Paleta:**
- `index.html` / `presentes.html`: fundo `#048279` (verde-azulado do convite)
- `amigos.html`: fundo `#B8A0D4` (lilás)
- Cards: `#f5f0ea` (creme) ou `#EDE6F5` (lilás claro na página de amigos)

**Fontes (Google Fonts + sistema):**
- Nomes: `'Snell Roundhand', 'Pinyon Script', cursive` — Snell Roundhand é fonte Apple; Pinyon Script é o fallback para Android
- Corpo elegante: `'Cormorant Garamond'`
- Labels/textos auxiliares: `'Jost'`

**Elementos decorativos:** SVGs inline de coração, laço, estrela e faíscas ao redor dos nomes no hero do `index.html`.

**Nomes:** exibidos como "Giovanna & Caio" (Giovanna primeiro).

---

## Sistema de secrets (GitHub Actions)

Dados sensíveis são **placeholders no HTML** injetados em build time pelo workflow.  
O deploy roda apenas no push para `main`.  
**GitHub Pages deve estar configurado como source: GitHub Actions** (não "Deploy from a branch").

| Secret | Usado em | Descrição |
|---|---|---|
| `PIX_KEY` | `presentes.html` | Chave PIX |
| `PIX_NAME` | `presentes.html` | Nome do favorecido PIX |
| `ADDRESS` | `presentes.html` | Endereço para envio de presentes |
| `RECEPTION_ADDRESS` | `index.html`, `evento.ics` | Endereço da recepção do casamento |
| `RECEPTION_MAPS_URL` | `index.html` | Link Google Maps da recepção |
| `GUEST_LIST` | `index.html` | JSON array com nomes dos convidados da lista fechada |
| `FORM_ENDPOINT` | `index.html`, `amigos.html` | Endpoint Formspree para receber confirmações |
| `PARTY_MAPS_URL` | `amigos.html` | Link Google Maps da festa de amigos |

**Formato do GUEST_LIST:**
```json
["Ana Maria Nolasco","Tharsis Paranhos","Luciana Santos","Athos Nolasco","Camila Gomide","Dante Nolasco","Cristina Nolasco","Luiz Carlos Lopes","Adolfo Lara","Adolfo Mendes","Sônia Lara"]
```

> ⚠️ **Bug conhecido:** O `sed` quebra se o valor de um secret contiver o caractere `|` (usado como delimitador) ou quebras de linha. Se o `RECEPTION_ADDRESS` ou outro endereço contiver `|`, o workflow falha com `unterminated s command`. Solução: garantir que os secrets não contenham `|`, ou migrar a injeção desses campos para Python (como foi feito e revertido — ver commit `4db882e`).

---

## Funcionalidades por página

### `index.html` — Casamento (09/07/2026)

- Hero com nomes em cursiva + elementos SVG decorativos + "Save the date"
- Countdown regressivo até 09/07/2026 09:30
- Grade de detalhes (2×2):
  - **Recepção**: `%%RECEPTION_ADDRESS%%` + horário 11h–14h + link Maps
  - **Traje**: Esporte fino
  - **Lista de presentes**: link para `presentes.html`
  - **Salvar na agenda**: botões Google Calendar e Apple/iCal (`evento.ics`)
- **RSVP com lista fechada:**
  - Campo de nome com **autocomplete** (dropdown filtra a partir de 2 chars)
  - Busca tolerante a acentos e capitalização
  - Se nome encontrado: mostra botões **Sim / Não**
  - Respostas enviadas via `fetch` ao Formspree com campo `pagina: 'familiar'` (implícito)
  - Mensagem de agradecimento customizada

### `presentes.html` — Lista de presentes

- Lista única (uma caixa, itens separados por linha fina): roupa de cama, toalhas, batedeira, processador, panelas, robô aspirador, taças de vinho, faqueiro, cota apartamento, cota lua de mel
- Botão Amazon (link direto para wishlist)
- Seção de endereço para envio (`%%ADDRESS%%`)
- Seção PIX com botão "Copiar"

### `amigos.html` — Festa de amigos (11/07/2026)

- Hero com mesmos nomes + decorações SVG
- Countdown até 11/07/2026 19:30
- Grade de detalhes:
  - **Local**: `%%NOME_DO_LOCAL%%` + `%%ENDEREÇO%%` + horário 19h–00h + link Google Maps (`%%PARTY_MAPS_URL%%`)
  - **Traje**: "Club chic — a balada dos seus sonhos de criança. Só evite o branco e o prata!" + link Pinterest (https://pin.it/11fLOyR3L)
  - **Salvar na agenda**: botões Google Calendar e iCal (`amigos.ics`)
- Seções informativas em cursiva (`.after-section`), nesta ordem:
  1. **E o jogo?** — Se houver jogo do Brasil, teremos transmissão!
  2. **E o presente?** — Seu presente é sua presença!
  3. **E o bar?** — Bar com comanda individual e bolo para adoçar a noite.
  4. **E o after?** — Vai ter after sim! Em breve mais informações.
- **RSVP aberto (sem lista fechada):**
  - Campo de nome livre (sem validação)
  - Opções: **Sim, estarei lá!** / **Sim, com +1!** / **Não poderei ir**
  - Se "+1": pergunta o nome do acompanhante
  - Respostas enviadas ao Formspree com campo `pagina: 'amigos'`

---

## Deploy

```bash
# Para acionar deploy manualmente (após merge na main):
# GitHub → Actions → Deploy → Run workflow

# Para commitar e enviar alterações do worktree:
git add <arquivos>
git add -f .github/workflows/deploy.yml  # .github está no .gitignore do worktree
git commit -m "mensagem"
git push
# Se rejeitado por non-fast-forward:
git pull --rebase && git push
```

---

## Pendências conhecidas

- [ ] **Bug do `|` no sed**: se qualquer secret de endereço contiver `|`, o workflow quebra. Solução definitiva: migrar injeção dos campos de endereço para Python (ver commit `4db882e` que foi revertido — pode ser reaproveitado)
- [ ] `PARTY_MAPS_URL` ainda não cadastrado no GitHub (festa de amigos)
- [ ] `FORM_ENDPOINT` (Formspree) ainda não cadastrado — RSVP não está enviando respostas
- [ ] `RECEPTION_NAME` foi removido; `RECEPTION_ADDRESS` e `RECEPTION_MAPS_URL` ainda precisam ser cadastrados
- [ ] `amigos.ics` existe no repo principal mas pode precisar de atualização de horário (19h–00h) e injeção de endereço via workflow

---

## Prompt de handoff para nova sessão

```
Estou desenvolvendo um site de casamento estático hospedado em GitHub Pages.

**Repositório:** giovannaparanhos/wedding_registry  
**Branch de trabalho:** claude/infallible-proskuriakova-785abc  
**Worktree local:** /Users/giovannaparanhos/Documents/casamento/wedding_registry/.claude/worktrees/infallible-proskuriakova-785abc  
**URL do site:** https://giovannaparanhos.dev.br  

Toda a documentação do projeto está em `.ai-dev/HANDOFF.md` no worktree. Leia esse arquivo antes de qualquer coisa.

Regras importantes:
- Sempre trabalhe no worktree acima
- `.github/` está no `.gitignore` do worktree — use `git add -f` para o workflow
- Se o push for rejeitado, use `git pull --rebase && git push`
- Dados sensíveis são injetados via GitHub Actions secrets com placeholders `%%NOME%%`
- O deploy só roda no push para `main` — o branch de trabalho vai para PR

O bug mais recente: o workflow de deploy falha com `sed: unterminated s command` quando um secret de endereço contém o caractere `|`. A solução é garantir que os secrets não usem `|`, ou migrar a injeção desses campos para Python.
```
