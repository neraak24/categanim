# Agrupamento — integração com o hub

Chave do teste no hub: `agrup` (mude em `SITE_ID` no index.html, se preferir outro nome).

## 1. Banco (Supabase → SQL Editor)
A tabela `sessoes` restringe os valores de `site`. Inclua o novo:

```sql
alter table public.sessoes drop constraint if exists sessoes_site_check;
alter table public.sessoes add constraint sessoes_site_check
  check (site in ('cubos','adf','ao','jdneup','spanvisuo','adfwnv','compinst','hanoi','memcenas','agrup'));
```
(Se o nome da constraint for outro, confira em Table Editor → sessoes → constraints.)
As edge functions `validar-token`, `registrar-print` e `concluir-sessao` não precisam mudar.

## 2. Hub (index.html)
- Botão, na lista de testes da faixa etária desejada:
  `<button id="detalhe-site-agrup">Agrupamento</button>`
- Listener: `document.getElementById('detalhe-site-agrup').addEventListener('click', () => gerarLinkDetalhe('agrup'));`
- `SITE_URLS`: `agrup: 'https://SEU-SITE.netlify.app/',`
- `NOMES_TESTE`: `agrup: 'Agrupamento'`
- CSP do hub: acrescentar o endereço do site em `img-src`.
- Em `carregarSessoesDetalhe`, tratar `agrup` como `compinst` (`ehCompinst = s.site === 'compinst' || s.site === 'agrup'`)
  e, em `renderizarCardCompinst`, desviar no começo:

```js
function renderizarCardAgrup(etapaLabel, d) {
  const BASE = SITE_URLS.agrup + 'imagens/carta_';
  const grupo = arr => `<div style="display:flex;gap:4px;padding:4px;background:#f3ebdf;border-radius:8px">` +
    (arr || []).map(n => n ? `<img src="${BASE}${n}.png" style="width:48px;height:48px;object-fit:contain">`
                           : `<div style="width:48px;height:48px;border:1px dashed #b9ab9b;border-radius:6px"></div>`).join('') + `</div>`;
  return `<div style="display:flex;flex-direction:column;gap:6px;align-items:center">${grupo(d && d.grupo1)}${grupo(d && d.grupo2)}</div>`;
}
```
e em `abrirResultadoCompinst`, usar `site === 'agrup' ? renderizarCardAgrup(p.etapa, p.dados_cubo) : renderizarCardCompinst(...)`.

## 3. O que é gravado
Cada clique em **Próximo** → `registrar-print` com `etapa: "Rodada N"` e `dados_cubo`:
`{ rodada, completa, grupo1: [c,c,c,c], grupo2: [c,c,c,c], tempo_rodada_s, tempo_total_s }`
(c = número da carta: 1 zebras, 2 cães, 3 aves voando, 4 elefante, 5 urso, 6 peixes, 7 gato, 8 pássaro no ninho; a posição no array = posição na tela, da esquerda para a direita).
Ao terminar (botão Finalizar ou 360 s) → se havia cartas já colocadas na rodada em andamento, ela é gravada como `"Rodada N (incompleta)"` (vazios = `null`), e `concluir-sessao` grava `{ motivo, rodadas_completas, rodada_incompleta, tempo_total_s }`.

## Tutorial
Antes da rodada 1 há um tutorial (não conta nos 360 s). Ao clicar em **Começar**, o resultado dele é gravado como `etapa: "Tutorial"` com
`{ tutorial: true, grupo1, grupo2, cartas_exemplo: [5,4,6,7], tempo_s }` (as 4 cartas do exemplo são colocadas pelo sistema: urso e elefante no grupo 1, peixes e gato no grupo 2).
Para não gravar o tutorial, mude `GRAVAR_TUTORIAL` para `false` no index.html. Se o teste for finalizado durante o tutorial, `concluir-sessao` grava `motivo: "finalizado_no_tutorial"`.

## 4. Testar sem hub
Abra `index.html?demo=1` (servido por http, não por duplo clique no arquivo): funciona igual, mas não valida nem grava nada (o que seria enviado aparece no console).
