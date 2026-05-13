# Sistema de Perfil com Menu Dropdown e Modal de Avatar

> **Guia completo:** código HTML + CSS + JavaScript puro, com explicação didática passo a passo.

---

## Índice

1. [Código HTML](#1-código-html)
2. [Código CSS](#2-código-css)
3. [Código JavaScript](#3-código-javascript)
4. [Como aprender com isso — Explicação Didática](#4-como-aprender-com-isso--explicação-didática)
5. [Próximos passos / Como evoluir](#5-próximos-passos--como-evoluir)

---

## 1. Código HTML

```html
<!--
  ESTRUTURA PRINCIPAL
  Tudo fica dentro de .user-profile.
  Ordem: ícone → menu dropdown → modal de avatar.
-->

<div class="user-profile">

  <!-- ── ÍCONE DE PERFIL ─────────────────────────────────────── -->
  <!--
    É a imagem circular clicável no topo da página.
    O id="profile-icon" é usado pelo JS para:
      - abrir/fechar o dropdown ao clicar
      - trocar o src quando um avatar for escolhido
  -->
  <img
    id="profile-icon"
    class="profile-icon"
    src="/img/avatar/avatar-default.png"
    alt="Foto de perfil"
  />

  <!-- ── MENU DROPDOWN ──────────────────────────────────────── -->
  <!--
    Começa escondido (sem a classe .show).
    O JS adiciona/remove .show ao clicar no ícone de perfil.
    position: absolute faz ele flutuar abaixo do ícone.
  -->
  <nav class="profile-menu" id="profile-menu" aria-label="Menu do usuário">

    <!-- Opção 1: link para a página de perfil -->
    <a class="menu-item" href="/profile">Meu perfil</a>

    <!-- Opção 2: botão que abre o modal de avatares -->
    <!--
      É um <button> (não um <a>) porque não navega: executa uma ação.
      A classe .avatar-trigger é o gatilho que o JS escuta.
    -->
    <button class="menu-item avatar-trigger" type="button">
      Mudar avatar
    </button>

    <!-- Opção 3: sair da conta -->
    <!--
      aria-label descreve a ação para leitores de tela,
      útil quando o texto sozinho não é suficientemente claro.
    -->
    <a
      class="menu-item"
      href="/logout"
      aria-label="Sair da conta"
    >Sair</a>

  </nav>

  <!-- ── MODAL DE AVATARES ──────────────────────────────────── -->
  <!--
    .modal-overlay cobre toda a tela com fundo escuro e blur.
    Começa escondido (display: none no CSS).
    Clicar no overlay (fora do card) fecha o modal.
  -->
  <div class="modal-overlay" id="modal-overlay">

    <!--
      .modal-center é o card branco centralizado.
      Clicar DENTRO dele não deve fechar o modal,
      por isso o JS verifica se o clique foi no overlay, não no card.
    -->
    <div class="modal-center">

      <h2 class="modal-title">Escolha um avatar</h2>

      <!--
        .avatar-grid organiza os avatares em grade (3 colunas).
        Cada botão tem data-avatar="N" — o JS lê esse atributo
        para saber qual avatar foi escolhido.
      -->
      <div class="avatar-grid">

        <button
          class="avatar-option"
          type="button"
          data-avatar="1"
          data-src="/img/avatars/avatar1.png"
          aria-label="Avatar 1"
        >
          <img src="/img/avatars/avatar1.png" alt="Avatar 1" />
        </button>

        <button
          class="avatar-option"
          type="button"
          data-avatar="2"
          data-src="/img/avatars/avatar2.png"
          aria-label="Avatar 2"
        >
          <img src="/img/avatars/avatar2.png" alt="Avatar 2" />
        </button>

        <button
          class="avatar-option"
          type="button"
          data-avatar="3"
          data-src="/img/avatars/avatar3.png"
          aria-label="Avatar 3"
        >
          <img src="/img/avatars/avatar3.png" alt="Avatar 3" />
        </button>

      </div><!-- /avatar-grid -->

      <!-- Botão para fechar o modal sem escolher nada -->
      <button class="modal-close" id="modal-close" type="button">
        Fechar
      </button>

    </div><!-- /modal-center -->
  </div><!-- /modal-overlay -->

</div><!-- /user-profile -->
```

---

## 2. Código CSS

```css
/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ÍCONE DE PERFIL
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */

/*
  .user-profile é o contêiner pai.
  position: relative faz o dropdown se posicionar
  em relação a ele, e não à janela.
*/
.user-profile {
  position: relative;
  display: inline-block; /* ocupa só o espaço do ícone */
}

.profile-icon {
  width: 44px;
  height: 44px;
  border-radius: 50%;          /* torna a imagem circular */
  border: 2px solid #d1d5db;   /* borda fina e discreta */
  object-fit: cover;           /* recorta a imagem sem distorcer */
  cursor: pointer;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

.profile-icon:hover {
  border-color: #6366f1;       /* bordinha roxa no hover */
  box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.15);
}


/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   MENU DROPDOWN
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */

/*
  display: none esconde o menu por padrão.
  position: absolute o faz flutuar sobre o conteúdo da página,
  sem empurrar outros elementos.
  top: calc(100% + 8px) posiciona logo abaixo do ícone (pai).
  right: 0 alinha pela direita.
*/
.profile-menu {
  display: none;
  position: absolute;
  top: calc(100% + 8px);
  right: 0;
  min-width: 180px;
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 10px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
  overflow: hidden;
  z-index: 100; /* fica acima do conteúdo da página */
}

/*
  Quando o JS adiciona a classe .show, o menu aparece.
  Usar display: flex permite empilhar os itens verticalmente.
*/
.profile-menu.show {
  display: flex;
  flex-direction: column;
}

/* Estilo compartilhado entre <a> e <button> do menu */
.menu-item {
  display: block;
  width: 100%;
  padding: 10px 16px;
  font-size: 0.92rem;
  color: #374151;
  text-decoration: none;
  text-align: left;
  background: none;
  border: none;
  cursor: pointer;
  transition: background 0.15s ease;
}

.menu-item:hover {
  background: #f3f4f6;
  color: #111827;
}


/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   MODAL COM BLUR (OVERLAY)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */

/*
  position: fixed faz o overlay cobrir TODA a janela,
  independentemente de o usuário ter rolado a página.
  inset: 0 é atalho para top/right/bottom/left: 0.

  backdrop-filter: blur() borra o conteúdo por BAIXO do overlay.
  background-color semi-transparente deixa o blur visível.

  display: none esconde por padrão; o JS muda para flex quando abre.
*/
.modal-overlay {
  display: none;
  position: fixed;
  inset: 0;
  background-color: rgba(15, 15, 20, 0.55);
  backdrop-filter: blur(6px);
  -webkit-backdrop-filter: blur(6px); /* Safari */
  z-index: 200; /* acima do dropdown */

  /* centraliza o card dentro do overlay */
  align-items: center;
  justify-content: center;
}

/* Quando o JS adiciona .show, o modal aparece como flex */
.modal-overlay.show {
  display: flex;
}


/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   CARD DO MODAL (CENTRO)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */

.modal-center {
  background: #ffffff;
  border-radius: 16px;
  padding: 28px 32px;
  width: 100%;
  max-width: 360px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.25);
  text-align: center;
}

.modal-title {
  font-size: 1.1rem;
  font-weight: 600;
  color: #111827;
  margin-bottom: 20px;
}


/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   GRADE DE AVATARES
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */

.avatar-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  justify-items: center;
  margin-bottom: 20px;
}

/*
  Cada opção é um <button> sem estilo padrão,
  transformado em círculo com border-radius: 50%.
*/
.avatar-option {
  background: none;
  border: 3px solid transparent;
  border-radius: 50%;
  padding: 3px;
  cursor: pointer;
  transition: border-color 0.18s ease, transform 0.18s ease;
}

.avatar-option img {
  width: 72px;
  height: 72px;
  border-radius: 50%;
  object-fit: cover;
  display: block;
}

.avatar-option:hover {
  border-color: #a5b4fc;
  transform: scale(1.05);
}

/*
  Quando o JS adiciona .selected, a borda fica roxa/cheia,
  indicando qual avatar foi escolhido.
*/
.avatar-option.selected {
  border-color: #6366f1;
  box-shadow: 0 0 0 2px rgba(99, 102, 241, 0.3);
}


/* ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   BOTÃO FECHAR
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ */

.modal-close {
  padding: 8px 24px;
  font-size: 0.9rem;
  color: #6b7280;
  background: #f3f4f6;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.15s ease;
}

.modal-close:hover {
  background: #e5e7eb;
  color: #111827;
}
```

---

## 3. Código JavaScript

```javascript
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// SELETORES — buscamos os elementos do DOM uma única vez
// e guardamos em variáveis para reutilizar ao longo do código.
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

const profileIcon   = document.getElementById('profile-icon');   // foto de perfil
const profileMenu   = document.getElementById('profile-menu');   // dropdown
const avatarTrigger = document.querySelector('.avatar-trigger'); // botão "Mudar avatar"
const modalOverlay  = document.getElementById('modal-overlay'); // fundo do modal
const modalClose    = document.getElementById('modal-close');   // botão "Fechar"
const avatarOptions = document.querySelectorAll('.avatar-option'); // todos os avatares


// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// RESTAURAR AVATAR SALVO (ao carregar a página)
// Se o usuário já escolheu um avatar em outra visita,
// o localStorage terá salvo o caminho. Carregamos aqui.
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

const savedAvatar = localStorage.getItem('selectedAvatarSrc');

if (savedAvatar) {
  // Atualiza o ícone do perfil com o avatar salvo
  profileIcon.src = savedAvatar;

  // Marca visualmente o avatar correto na grade
  avatarOptions.forEach(function(btn) {
    if (btn.dataset.src === savedAvatar) {
      btn.classList.add('selected');
    }
  });
}


// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// DROPDOWN — abrir e fechar ao clicar no ícone de perfil
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

// classList.toggle('show') adiciona .show se não existir,
// ou remove se já existir — perfeito para alternar o menu.
profileIcon.addEventListener('click', function(event) {
  event.stopPropagation(); // impede que o clique "escoe" para o document
  profileMenu.classList.toggle('show');
});


// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// FECHAR DROPDOWN ao clicar em qualquer lugar fora dele
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

document.addEventListener('click', function(event) {
  // contains() verifica se o clique foi DENTRO do .user-profile
  // Se não foi, remove a classe .show para fechar o menu.
  const userProfile = document.querySelector('.user-profile');
  if (!userProfile.contains(event.target)) {
    profileMenu.classList.remove('show');
  }
});


// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// FUNÇÕES AUXILIARES — abrir e fechar o modal
// Ter funções nomeadas facilita a leitura e reutilização.
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

function openModal() {
  modalOverlay.classList.add('show');
}

function closeModal() {
  modalOverlay.classList.remove('show');
}


// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// ABRIR MODAL ao clicar em "Mudar avatar"
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

avatarTrigger.addEventListener('click', function() {
  profileMenu.classList.remove('show'); // fecha o dropdown primeiro
  openModal();
});


// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// FECHAR MODAL — botão "Fechar"
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

modalClose.addEventListener('click', closeModal);


// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// FECHAR MODAL ao clicar no overlay (fundo escuro)
// Mas NÃO fechar se o clique for dentro do .modal-center (card branco).
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

modalOverlay.addEventListener('click', function(event) {
  // event.target é o elemento EXATAMENTE clicado.
  // Se for o overlay em si (e não o card dentro dele), fecha.
  if (event.target === modalOverlay) {
    closeModal();
  }
});


// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// SELECIONAR AVATAR
// Ao clicar num botão de avatar:
//   1. Remove .selected de todos os outros botões
//   2. Adiciona .selected no botão clicado
//   3. Lê o caminho da imagem via data-src
//   4. Atualiza o ícone de perfil
//   5. Salva no localStorage
//   6. Fecha o modal
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

avatarOptions.forEach(function(button) {
  button.addEventListener('click', function() {

    // Passo 1 — remove seleção de todos
    avatarOptions.forEach(function(btn) {
      btn.classList.remove('selected');
    });

    // Passo 2 — marca o clicado como selecionado
    button.classList.add('selected');

    // Passo 3 — lê o atributo data-src do botão
    // No HTML: data-src="/img/avatars/avatar1.png"
    // No JS:   button.dataset.src  → "/img/avatars/avatar1.png"
    const newSrc = button.dataset.src;

    // Passo 4 — atualiza o ícone de perfil visualmente
    profileIcon.src = newSrc;

    // Passo 5 — salva no localStorage para persistir entre visitas
    localStorage.setItem('selectedAvatarSrc', newSrc);

    // Passo 6 — fecha o modal automaticamente
    closeModal();
  });
});
```

---

## 4. Como aprender com isso — Explicação Didática

### 4.1 O papel de cada tecnologia

Pense nas três tecnologias como três profissionais diferentes construindo uma casa:

| Tecnologia | Responsabilidade neste componente |
|---|---|
| **HTML** | A estrutura — define quais elementos existem (ícone, menu, modal, botões, imagens) |
| **CSS** | A aparência — define cor, forma, posição, visibilidade e animações |
| **JavaScript** | O comportamento — responde a cliques e muda o HTML/CSS dinamicamente |

Nenhuma das três substitui a outra. O modal existe no HTML, é estilizado pelo CSS e aberto pelo JS.

---

### 4.2 Menu dropdown: o que é e como o JS o controla

Um **dropdown** é um menu que fica escondido e aparece ao interagir com um elemento (aqui, o ícone de perfil).

A estratégia usada:

```css
/* O menu começa escondido */
.profile-menu { display: none; }

/* Com a classe .show, ele aparece */
.profile-menu.show { display: flex; }
```

```javascript
// classList.toggle adiciona .show se não existir, remove se já existir
profileMenu.classList.toggle('show');
```

**Analogia:** Imagine um interruptor de luz. `classList.toggle('show')` é o interruptor — cada clique alterna entre ligado e desligado.

Por que `classList.toggle` e não mudar o `display` direto no JS?

- **Separação de responsabilidades:** o JS não precisa saber como o menu se parece (isso é do CSS). Ele só diz "está ativo ou não".
- **Mais fácil de manter:** se quiser mudar a animação, você muda só o CSS, sem tocar no JS.

---

### 4.3 Modal centralizado: como o CSS o posiciona

```css
.modal-overlay {
  position: fixed;  /* cobre a janela toda, não o documento */
  inset: 0;         /* top: 0; right: 0; bottom: 0; left: 0 */
  display: flex;
  align-items: center;     /* centraliza verticalmente */
  justify-content: center; /* centraliza horizontalmente */
}
```

O overlay (`position: fixed; inset: 0`) age como um "vidro" que cobre 100% da janela visível, mesmo se o usuário tiver rolado a página.

O card (`.modal-center`) fica centralizado por `flexbox` no overlay — sem precisar de `position: absolute`, `top: 50%` ou `transform: translate`. Mais simples e moderno.

> **Por que `position: fixed` e não `absolute`?**
> `absolute` seria relativo ao elemento pai mais próximo com `position` definido. `fixed` é sempre relativo à **janela do navegador** — perfeito para modais que precisam cobrir tudo.

---

### 4.4 `backdrop-filter: blur` — o efeito de fundo borrado

```css
.modal-overlay {
  background-color: rgba(15, 15, 20, 0.55); /* fundo semitransparente */
  backdrop-filter: blur(6px);               /* borra o que está ATRÁS */
  -webkit-backdrop-filter: blur(6px);       /* prefixo para Safari */
}
```

`backdrop-filter` aplica um filtro no conteúdo **por baixo** do elemento, não nele mesmo. É como uma lente de vidro fosco.

Para ele funcionar, o fundo do overlay precisa ser semitransparente (com `rgba` e alpha menor que 1) — senão o conteúdo por baixo ficaria completamente coberto e o blur não seria visível.

---

### 4.5 Atributos `data-*` e como o JS os lê

No HTML você pode criar atributos personalizados prefixando com `data-`:

```html
<button data-avatar="1" data-src="/img/avatars/avatar1.png">...</button>
```

No JavaScript, você acessa via `element.dataset`:

```javascript
button.dataset.avatar  // → "1"
button.dataset.src     // → "/img/avatars/avatar1.png"
```

**Por que usar `data-src` em vez de pegar o `src` da `<img>` dentro do botão?**

- Mais explícito e confiável. O `dataset` é um contrato claro: "este botão representa este avatar neste caminho".
- Se você quiser mudar o tamanho ou o formato da imagem exibida, não precisa alterar o JS.

---

### 4.6 Como o `localStorage` funciona aqui

O `localStorage` é um "caderninho" que o navegador guarda para o site. Ele sobrevive ao fechar e reabrir a aba.

```javascript
// Salvar
localStorage.setItem('selectedAvatarSrc', '/img/avatars/avatar2.png');

// Ler
const src = localStorage.getItem('selectedAvatarSrc'); // → "/img/avatars/avatar2.png"

// Remover
localStorage.removeItem('selectedAvatarSrc');
```

No nosso código, salvamos o caminho da imagem escolhida. Ao carregar a página (no topo do JS), verificamos se existe algum valor salvo e, se existir, já aplicamos o avatar correto.

**Limitações do `localStorage`:**
- Funciona só no mesmo navegador e dispositivo.
- Não é seguro para dados sensíveis (é acessível via console).
- Não funciona entre usuários diferentes.

Por isso, para um sistema real com múltiplos dispositivos, você vai querer o Supabase (veja seção 5).

---

### 4.7 Plano de estudo prático

#### Fase 1 — Só o dropdown (1–2 horas)

1. Crie um HTML com um botão e um `<div>` oculto abaixo dele.
2. Faça o CSS esconder o `<div>` por padrão.
3. Com JS, use `classList.toggle` para mostrar/esconder ao clicar no botão.
4. Adicione o evento de "clicar fora para fechar".

#### Fase 2 — Só o modal (1–2 horas)

1. Crie um HTML com um overlay e um card centralizado.
2. Faça o CSS posicionar o overlay com `position: fixed; inset: 0`.
3. Adicione o `backdrop-filter: blur`.
4. Com JS, abra o modal com um botão e feche com outro ou clicando no overlay.

#### Fase 3 — Só a seleção de avatar (1 hora)

1. Crie uma grade de 3 imagens (podem ser placeholders do `via.placeholder.com`).
2. Ao clicar em uma, adicione a classe `.selected` nela e remova das outras.
3. Pegue o `src` da imagem clicada e mostre em outro lugar da página.

#### Fase 4 — Juntar tudo

Combine o que fez nas três fases. Você vai perceber que cada parte já funciona isolada — basta conectá-las.

---

### 4.8 Boas práticas observadas neste código

- **Separação de responsabilidades:** CSS controla a aparência; JS controla o comportamento. JS nunca muda `display` diretamente — só adiciona/remove classes.
- **IDs para unicidade, classes para estilo:** `id="modal-overlay"` é usado pelo JS para identificar o elemento. `.show` é uma classe de estado usada pelo CSS.
- **`stopPropagation` com cuidado:** usado no clique do ícone para que o evento não "borbulhe" até o `document` e feche o menu imediatamente.
- **Acessibilidade básica:** `aria-label` nos links e botões, `alt` nas imagens.
- **Sem elementos repetidos:** um único modal, um único menu, um único ícone.

---

## 5. Próximos passos / Como evoluir

### 5.1 Integração com Supabase

O `localStorage` é ótimo para protótipos, mas não sincroniza entre dispositivos. Com Supabase, você persiste o avatar no banco de dados.

**Estrutura sugerida na tabela `profiles`:**

```sql
-- Na tabela profiles (já criada pelo Supabase Auth se você usar RLS)
ALTER TABLE profiles
  ADD COLUMN avatar_id   INTEGER,      -- ID numérico do avatar (1, 2, 3...)
  ADD COLUMN avatar_url  TEXT;         -- URL completa (opcional, mas prático)
```

**Salvar o avatar ao selecionar:**

```javascript
// Após o usuário escolher um avatar
const { error } = await supabase
  .from('profiles')
  .update({ avatar_url: newSrc, avatar_id: avatarId })
  .eq('id', userId);
```

**Carregar o avatar ao fazer login:**

```javascript
const { data: profile } = await supabase
  .from('profiles')
  .select('avatar_url')
  .eq('id', userId)
  .single();

if (profile?.avatar_url) {
  profileIcon.src = profile.avatar_url;
}
```

**Dica:** prefira salvar `avatar_id` (número inteiro) ao invés de `avatar_url` (string longa). Isso facilita mudar o domínio das imagens no futuro sem precisar atualizar cada linha do banco.

---

### 5.2 Outras evoluções possíveis

| Melhoria | Como fazer |
|---|---|
| **Animação de entrada do modal** | Adicionar `@keyframes` no CSS e `animation` em `.modal-overlay.show` |
| **Fechar modal com tecla Esc** | `document.addEventListener('keydown', e => { if (e.key === 'Escape') closeModal() })` |
| **Mais avatares / scroll na grade** | Adicionar `max-height` e `overflow-y: auto` na `.avatar-grid` |
| **Upload de foto própria** | `<input type="file">` + Supabase Storage para fazer upload e salvar a URL |
| **Reutilizar o modal** | Transformar em uma função `openAvatarModal()` chamável de qualquer parte do site |
| **Tooltip no hover do ícone** | Pseudo-elemento `::after` no CSS com `content: "Perfil"` |

---

> **Resumo em uma frase:** HTML define o que existe, CSS define como parece e onde fica, JavaScript define o que acontece quando o usuário age. Dominar esses três papéis é a base de qualquer interface web.
