# GPC Sistema Analítico — Pacote de deploy

## O que subir no GitHub Pages (dash.solucoesr2.com.br)

Todos os arquivos **na raiz** do repositório (mesmo nível do CNAME):

| Arquivo | O que é |
|---------|---------|
| **`index.html`** | Sistema principal. Sobrescreve o atual. |
| **`filiais.json`** | Índice de filiais e bases. |
| **`consolidado.json`** | KPIs agregados do grupo. |
| **`dados-atp.json`** | Dados granulares da filial ATP. |
| **`snapshots.json`** | Índice de snapshots históricos (hoje vazio). |

Esses 5 arquivos vão para o GitHub. O sistema carrega todos via `fetch()`.

---

## O que NÃO subir (use local)

| Arquivo | Para que serve |
|---------|---------------|
| **`seed-firebase.html`** | Abrir uma vez no seu PC para popular o Firestore. Depois pode apagar. |

---

## O que é referência (não sobe)

| Arquivo | Para que serve |
|---------|---------------|
| **`firestore.rules`** | Texto das regras de segurança. Você cola no Firebase Console manualmente. |
| **`FIREBASE_SETUP.md`** | Documentação completa do processo. Para consulta. |
| **`docs/`** | Documentação técnica do sistema. |

---

## Ordem de execução

### 1. Publicar regras temporárias no Firestore

Firebase Console → Firestore → Regras → colar:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

Publicar.

### 2. Rodar o seed

Abrir `seed-firebase.html` no navegador (2 cliques, local mesmo), logar com seu email/senha do Firebase, clicar os 3 botões: **Popular perfis** → **Criar admin** → **Verificar**.

### 3. Publicar regras de produção

Voltar no Firestore → Regras → apagar e colar o conteúdo de `firestore.rules` (só o bloco "REGRAS DE PRODUÇÃO", sem os comentários).

Publicar.

### 4. Subir no GitHub Pages

Subir estes 5 arquivos:
- `index.html`
- `filiais.json`
- `consolidado.json`
- `dados-atp.json`
- `snapshots.json`

### 5. Testar

Abrir `dash.solucoesr2.com.br` em janela anônima (Ctrl+Shift+N).
Logar com `r2@solucoesr2.com.br` + senha.

---

## Se quebrar

Abrir o console do navegador (F12 → Console) e verificar as mensagens em vermelho.

Causas prováveis de erro:
- **404 em JSON**: arquivo não foi subido pro GitHub Pages
- **Permission denied**: regras do Firestore incorretas (voltar para regras temporárias)
- **Auth error**: email/senha incorretos, ou admin não existe no Firestore (rodar o seed)

Qualquer erro, me envia print do console.
