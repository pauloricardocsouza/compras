# GPC Sistema Analítico — Estrutura de Deploy

A partir desta versão, o sistema é composto por **2 arquivos**:

| Arquivo | Tamanho | O que contém |
|---------|---------|--------------|
| `index.html` | ~242 KB | Toda a interface, CSS e lógica |
| `dados.json` | ~10 MB  | Apenas os dados do período |

## Como atualizar os dados (sem mexer no código)

1. Substitua **apenas** o `dados.json` no GitHub Pages (deixa o `index.html` intocado)
2. O navegador busca `dados.json` automaticamente ao abrir
3. Cache HTTP padrão se aplica — força refresh se quiser ver imediatamente

## Como atualizar o código (sem mexer nos dados)

1. Substitua **apenas** o `index.html`
2. O `dados.json` continua sendo carregado normalmente

## Vantagens

- **HTML 43× menor** (de 10,3 MB para 242 KB) — carrega instantâneo
- **Cache do navegador funciona melhor** — código fica em cache mesmo quando dados mudam
- **Atualizar dados não exige redeploy do HTML inteiro**
- **Erros de carregamento são visíveis** — tela de loading com hash dos dados

## Estrutura interna do dados.json

```json
{
  "_meta": {
    "gerado_em": "2026-04-22T10:32:56",
    "hash": "9e0443b0",
    "versao": "1.0",
    "tamanho_bytes": 10584769
  },
  "D": {
    "meta": {...},
    "produtos": [...],
    "fornecedores": [...],
    ...
  }
}
```

O hash `_meta.hash` aparece embaixo do loader na tela inicial — útil para confirmar visualmente que a versão correta foi carregada.

## Compatibilidade reversa

Se `dados.json` falhar (404, erro de rede etc), o sistema tenta carregar do bloco `<script id="GD">` embutido no HTML — mas hoje esse bloco está vazio (`{}`). Se quiser manter um fallback embutido, basta colar dados antigos dentro dele.

## Servir localmente para testar

```bash
cd pasta_com_arquivos
python3 -m http.server 8000
# Acesse http://localhost:8000
```

Não funciona com `file://` diretamente no navegador por causa do CORS — precisa de servidor HTTP.
