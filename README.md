# Central de Conteúdo R1

Painel sobre a pasta mãe do Drive do Grupo R1: acha o vídeo, decupa por
minutagem, toca a fila de edição e guarda o procedimento.

Roda também dentro do painel da Turnê, em `/conteudo` — este repositório é a
mesma ferramenta como projeto próprio, pra quem preferir separar o deploy.

## As cinco abas

| Aba | Pra quem | O que faz |
|---|---|---|
| **Central** | todo mundo | As 9 pastas de tema, montadas pelo sistema (pasta do Drive > tag > nome/observação). A `00. Fluxo Diário` lista o que ainda não tem trecho marcado. |
| **Decupagem** | minerador | Acha o vídeo no acervo (17.787), assiste embutido, marca início/fim, headline, editoria, produto, redes e status. |
| **Painel** | editor | Todos os trechos numa tabela. Status e link do editado editáveis na própria linha. |
| **Minutagem RGV** | todo mundo | Busca nas transcrições dos eventos. |
| **POP** | todo mundo | O procedimento: mapa da pasta, papéis, as 5 etapas com regra de saída, ritmo e métricas. |

## Subir na Vercel

1. **Import Git Repository** → este repositório. *Framework Preset* = **Other**,
   *Output Directory* = **`public`**, sem build.
2. Uma variável de ambiente: **`DATABASE_URL`**, a mesma string do Neon que o
   `painel-turne-mdv` usa (com `-pooler` no host).
3. Deploy.

Só isso. Na primeira chamada a `/api/conteudo` as tabelas nascem sozinhas e os
dados que estão hoje no Netlify entram junto. Um marcador na tabela `config`
(`conteudo_importado`) garante que a importação não se repita.

As tabelas desta ferramenta começam com `conteudo_` e **não encostam nas do
painel** — dividir o mesmo banco é seguro.

Se a importação automática falhar (Netlify fora do ar, por exemplo), a API abre
vazia e o caminho manual continua valendo:

```bash
node ferramentas/importar-conteudo.mjs                       # busca do Netlify
node ferramentas/importar-conteudo.mjs dados/backup-2026-09-23.json
```

⚠️ `api/conteudo.js` foi escrito junto com a migração e **não foi testado contra
um banco real**. Depois do primeiro deploy, confira um `GET /api/conteudo` e um
`POST` de ida e volta antes de confiar.

Enquanto isto não sobe, a versão no ar é
<https://central-de-conteudo-r1.netlify.app> (mesma tela, dados no Netlify Blobs).

## A API

```
GET  /api/conteudo                      -> { cortes: [...], videos: [...] }
POST /api/conteudo {op:"set", col, doc} -> { ok: true }   // doc.id obrigatório
POST /api/conteudo {op:"del", col, id}  -> { ok: true }   // vídeo apagado leva os trechos
```

`col` é `"cortes"` ou `"videos"`. Uma linha por trecho: dois mineradores
marcando ao mesmo tempo não se sobrescrevem.

**Campos de um corte** — `id` · `brutoId` · `brutoNome` · `brutoLink` · `tcIn` ·
`tcOut` · `dur` (segundos) · `headline` · `minutadoPor` · `editoria` · `produto` ·
`obs` · `responsavel` · `rede` · `status` · `linkEditado` · `nota` · `desempenho` ·
`dataPost`

**status**: (vazio) · `DECUPADO` · `EM EDIÇÃO` · `EDITADO` · `POSTADO`
**nota**: `VIRAL` · `BOM` · `MÉDIO` · `FRACO`
**produto**: `RGV` · `CLUBE R1` · `DONO COM DONO` · `DE FRENTE COM RICARDO` · `DIA A DIA` · `FAMILIA` · `PALESTRAS` · `PODCASTS`
**rede**: várias por trecho, separadas por vírgula

## Arquivos

```
public/index.html      o app inteiro — HTML, CSS e JS num arquivo só, sem build
public/acervo.json     17.787 vídeos do Drive: id, título, pasta, data, tamanho
public/minutagem.html  o buscador de transcrições (12,5 MB, carrega só quando a aba abre)
api/conteudo.js        a API, no Postgres, com as tabelas se montando sozinhas
ferramentas/varrer-drive.py        regera o acervo varrendo a árvore do Drive
ferramentas/importar-conteudo.mjs  importação manual, se a automática falhar
dados/backup-*.json    export dos dados no dia da migração
```

## Manutenção

**O acervo é uma foto** tirada do Drive em 23/09/2026. Vídeo novo não aparece
sozinho:

```bash
python3 ferramentas/varrer-drive.py
```

Depois regere o `acervo.json` (id, título, pasta, data) e **suba o número da
versão** no `fetch("acervo.json?v=N")` dentro de `public/index.html` — o header é
`max-age=3600` e sem isso o navegador serve o arquivo velho por uma hora. A
varredura usa `drive.google.com/embeddedfolderview`, que dispensa credencial
porque a pasta mãe está compartilhada por link.

**Mapa de produto incompleto.** `MAPA_PRODUTO` no `index.html` deduz o produto
pela pasta do Drive. Onze pastas ainda não têm produto definido — entre elas
`[FD] CNE` (3.113 vídeos), `[FD] RGV PROCESSOS` (908) e `[FD] EXECUTIVOS` (519).
Nenhuma pasta aponta pra `FAMILIA`.

**Minutagem RGV** é cópia estática de minutagem.netlify.app; não se atualiza
sozinha.

## O que a varredura do Drive mostrou

Das 17.787 peças de vídeo, **17.760 estão paradas na `00. FLUXO DIÁRIO`**. As
nove pastas de tema somam 27. A entrada não é entrada: é o acervo inteiro
esperando mineração. É o gargalo que o POP descreve, com número.
