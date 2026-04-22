# Firebase — Setup e ativação

Este documento explica como plugar o Firebase real no sistema, passo a passo.

## Pré-requisitos (já feitos)

- ✅ Projeto Firebase `comercial-3029f` criado
- ✅ Authentication habilitado (Email/Senha)
- ✅ Cloud Firestore criado (região `southamerica-east1`)
- ✅ App Web registrado (`firebaseConfig` obtido)
- ✅ Usuário admin criado em Authentication → `r2@solucoesr2.com.br` (UID `cZLx89WTkgVhzW2dRu3ruLRxQwh2`)

## O que o sistema já está fazendo

O `index.html` entregue agora tem:

- **SDK Firebase carregado via CDN** (sem build step necessário)
- **`AUTH_MODE = 'firebase'`** (trocado de `'mock'`)
- **`firebaseConfig`** embutido no `<head>` antes do script principal
- **Adapter de autenticação** (`_doLogin`, `_getUsuarios`, `_getPerfis`, `_saveUsuario`, etc.) que delega para Firestore
- **Cache de 30s em memória** para usuários e perfis (reduz custo de reads)
- **`_auditLog`** escreve em `auditLog` do Firestore (fire-and-forget)
- **Modal de criar usuário** agora pede o UID do Firebase (opção B que você escolheu)

Mas **nada disso funciona ainda** porque o Firestore está vazio. Precisa popular.

---

## Setup em 6 passos

### Passo 1: Colocar regras temporárias no Firestore

O Firestore vem por padrão com regras que bloqueiam tudo (modo produção).

Acesse: **Firebase Console → Firestore Database → Regras**

Cole estas regras temporárias (vamos trocar depois):

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

Clique **Publicar**.

> ⚠ Isso permite que qualquer usuário logado leia e escreva em qualquer coleção. **Só mantenha por alguns minutos**, durante o seed.

### Passo 2: Abrir o seed-firebase.html localmente

Abra o arquivo **`seed-firebase.html`** entregue no seu navegador (clicar duas vezes / arrastar para o Chrome). Não precisa subir pro servidor — pode ser local mesmo.

### Passo 3: Fazer login

No topo do `seed-firebase.html`:
- Email: `r2@solucoesr2.com.br`
- Senha: a senha temporária que você definiu no Firebase Console quando criou o usuário

Clicar **Logar**. Deve aparecer "✓ Logado".

### Passo 4: Rodar os 3 botões do seed

1. **Popular perfis** — cria os 3 documentos em `perfisTemplate/` (admin, gestor, visualizador)
2. **Criar admin no Firestore** — cria `usuarios/cZLx89WTkgVhzW2dRu3ruLRxQwh2` com os dados do admin
3. **Verificar** — confirma que tudo está OK (cria um log de auditoria de teste)

Se algum falhar, ver o log preto na tela para diagnóstico.

### Passo 5: Trocar para regras de produção

Volta no **Firebase Console → Firestore → Regras** e substitui pelas regras de produção (arquivo `firestore.rules` entregue):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isSignedIn() { return request.auth != null; }
    function isAdmin() {
      return isSignedIn() &&
             exists(/databases/$(database)/documents/usuarios/$(request.auth.uid)) &&
             get(/databases/$(database)/documents/usuarios/$(request.auth.uid)).data.perfil == 'admin';
    }

    match /usuarios/{uid} {
      allow read: if isSignedIn() && (request.auth.uid == uid || isAdmin());
      allow write: if isAdmin();
    }
    match /perfisTemplate/{sigla} {
      allow read: if isSignedIn();
      allow write: if isAdmin();
    }
    match /auditLog/{logId} {
      allow create: if isSignedIn();
      allow read: if isAdmin();
      allow update, delete: if false;
    }
    match /configuracoes/{base} {
      allow read: if isSignedIn();
      allow write: if isAdmin();
    }
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

Publicar.

### Passo 6: Testar o sistema de verdade

1. Subir o novo `index.html` para o GitHub Pages (sobrescrever o antigo)
2. Abrir `dash.solucoesr2.com.br` em janela anônima
3. Fazer login com `r2@solucoesr2.com.br` + senha
4. Validar:
   - Topbar mostra "Ricardo (R2 Soluções)"
   - Menu "Administração" aparece
   - Na Administração, cards "Usuários", "Perfis", "Auditoria" carregam
   - Ir em Auditoria → deve ter pelo menos 1 log (o `seed_verify` + seu `login_ok`)

Se tudo OK, **o sistema está em produção no Firebase**.

### Passo 7 (limpeza)

- Deletar `seed-firebase.html` do servidor (se você tiver subido por engano)
- Pode manter localmente como referência

---

## Como criar novos usuários daqui pra frente (opção B)

Fluxo para o admin criar novos acessos:

1. No **Firebase Console → Authentication → Users → Add user**, criar email + senha temporária
2. Copiar o **User UID** que o Firebase gera
3. No sistema (dash.solucoesr2.com.br), logado como admin, ir em **Administração → Adicionar acesso**
4. Preencher:
   - **User UID** (colar do passo 2)
   - Email (mesmo do Firebase)
   - Nome completo
   - Perfil (admin / gestor / visualizador)
   - Páginas e filiais permitidas (opcional, se quiser customizar)
5. Salvar

Enviar ao novo usuário: email + senha temporária + link do sistema. No primeiro login, o sistema vai pedir para trocar a senha.

### Alternativas consideradas

- **Opção A** (criação pelo sistema): exigiria Cloud Functions (plano Blaze pago). Descartado.
- **Opção C** (tudo no console): muito manual, sem UI amigável. Descartado.

Se no futuro o volume de usuários crescer muito, migramos para opção A.

---

## Troubleshooting

### "Permission denied" ao criar admin

- Você publicou as regras de produção **antes** de rodar o seed. Volte para as regras temporárias, rode o seed, e depois republique as de produção.

### Login falha com "auth/invalid-credential"

- Email ou senha incorretos
- Ou: você criou o admin em um projeto diferente do que está configurado no `firebaseConfig`

### Mensagem "Usuário autenticado mas não cadastrado no sistema"

- Firebase Auth criou o usuário, mas o documento em `usuarios/{uid}` não existe no Firestore
- Solução: admin precisa criar o doc manualmente no Firestore, **ou** admin usa a UI "Adicionar acesso" passando o UID

### Auditoria vazia após ativar

- Pode ser cache (aguarda 15s e atualiza a página)
- Ou: regras de segurança bloqueando. Confirmar que `auditLog` tem `allow create: if isSignedIn()`

### Sessão expira muito rápido

- O sistema tem sessão local de 30 dias
- Firebase Auth tem sessão padrão, configurada como `LOCAL` no init (persiste entre abas/fechamentos)

---

## Custos esperados

Plano **Spark (gratuito)** cobre:

- 50k reads/dia (usuários, perfis, audit log)
- 20k writes/dia
- 20k deletes/dia
- 1 GB armazenamento

Para o GPC com ~20 usuários ativos, estimativa diária:

- Logins: ~40/dia → 40 reads
- Navegação (cache 30s): ~500 reads/dia
- Auditoria writes: ~500/dia
- Leitura de logs (só admin, 1-2x/dia): ~1000 reads

**Total: ~2k reads/dia, 500 writes/dia → dentro do free tier.**

Se ultrapassar: migrar para plano Blaze custa ~R$ 1,00/mês com esse uso.

---

## O que falta (backlog Firebase)

- Configurações sincronizadas em `configuracoes/{base}` (etapa 9 original)
- Password reset via email (hoje só admin reseta)
- Multi-factor authentication (MFA) opcional para perfil admin

Se quiser, incluo em próximas rodadas.
