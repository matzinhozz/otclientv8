# HTML/CSS Engine — OTClientV8

Sistema de renderização HTML/CSS integrado ao OTClientV8, permitindo criar UIs com HTML e estilizar com CSS.

## Sumário

1. [Arquitetura](#arquitetura)
2. [Início Rápido](#início-rápido)
3. [Atributos HTML Suportados](#atributos-html-suportados)
4. [Propriedades CSS Suportadas](#propriedades-css-suportadas)
5. [Data Binding (*atributos)](#data-binding)
6. [Diretivas de Fluxo (*if, *for)](#diretivas-de-fluxo)
7. [Eventos](#eventos)
8. [API Lua](#api-lua)
9. [API C++](#api-c)
10. [Exemplos Práticos](#exemplos-práticos)

---

## Arquitetura

```
┌──────────────────────────────────────────────────────┐
│  Lua Module (Controller)                             │
│  controller.lua → html.lua → test_html/              │
├──────────────────────────────────────────────────────┤
│  C++ HTML Engine                                     │
│  htmlparser  → parses HTML string                    │
│  cssparser   → parses CSS stylesheets                │
│  htmlnode    → DOM tree (HtmlNode)                   │
│  htmlmanager → g_html singleton, widget creation     │
│  queryselector → CSS selector engine                 │
├──────────────────────────────────────────────────────┤
│  C++ UI Layer                                        │
│  uiwidgethtml.cpp   → HTML widget rendering          │
│  uilayoutflexbox.cpp → Flexbox layout engine         │
│  uiwidget.cpp       → Block/Inline/Table layout      │
├──────────────────────────────────────────────────────┤
│  Data Styles                                         │
│  html.css    → global stylesheet (auto-loaded)       │
│  custom.css  → user overrides (auto-loaded)          │
│  html.otui   → UIHTML widget base style              │
└──────────────────────────────────────────────────────┘
```

### Fluxo de Carregamento

1. `Controller:loadHtml("arquivo.html")` → `g_html.load(moduleName, path, parent)`
2. Parser lê HTML → árvore DOM (HtmlNode)
3. Parser CSS aplica estilos (global + inline `<style>` + atributos)
4. `htmlmanager` cria widgets OTClient para cada nó
5. `html.lua` processa `*text`, `*if`, `*for`, eventos, etc.
6. Widgets renderizam via `uiwidgethtml.cpp` com layout engine

---

## Início Rápido

### Criar um módulo HTML mínimo

**1. Criar `modules/meu_modulo/meu_modulo.otmod`:**
```lua
Module
  name: meu_modulo
  description: Exemplo HTML
  author: Dev
  autoLoad: true
  autoLoadPriority: 9999
  scripts: [ meu_modulo.lua ]
  @onLoad: MeuModulo:onInit()
```

**2. Criar `modules/meu_modulo/meu_modulo.lua`:**
```lua
MeuModulo = Controller:new()

function MeuModulo:onInit()
    self:loadHtml('janela.html')
end
```

**3. Criar `modules/meu_modulo/janela.html`:**
```html
<style>
  .box {
    background-color: rgb(40, 40, 80);
    width: 300px;
    height: 200px;
    text-align: center;
    border: 1 #4444aa;
  }
</style>

<script type="text">
  self.contador = 0
</script>

<html>
<div class="box" placement="center">
  <label *text="'Contador: ' .. tostring(self.contador)" />
  <button onclick="self.contador = self.contador + 1">+1</button>
</div>
</html>
```

### Estilos globais

Os arquivos `data/styles/html.css` e `data/styles/custom.css` são carregados automaticamente pelo `init.lua`. Use `custom.css` para sobrescrever estilos de widgets nativos (ex: padding, margens).

---

## Atributos HTML Suportados

### Elementos HTML → Widgets OTClient

| Tag HTML        | Widget OTClient     |
|-----------------|---------------------|
| `<div>`         | UIHTML (container)  |
| `<span>`        | UIHTML (inline)     |
| `<label>`       | UILabel             |
| `<button>`      | UIButton            |
| `<input>`       | UITextEdit / UICheckBox / UIComboBox |
| `<img>`         | UIImageView         |
| `<textarea>`    | UITextEdit          |
| `<table>`       | Table layout        |
| `<tr>`          | TableRow            |
| `<td>`, `<th>`  | TableCell           |
| `<select>`      | UIComboBox          |
| `<checkbox>`    | UICheckBox          |
| `<link>`        | Carrega CSS externo |
| `<style>`       | CSS inline          |
| `<script>`      | Lua (executa no controller) |
| `<html>`        | Container raiz      |

### Atributos de Posicionamento

| Atributo      | Descrição                               |
|---------------|-----------------------------------------|
| `placement`   | `"center"` centraliza na tela           |
| `position`    | `"absolute"` posiciona relativo ao pai  |

---

## Propriedades CSS Suportadas

### Layout

| Propriedade      | Valores                          |
|------------------|----------------------------------|
| `display`        | `none`, `block`, `inline`, `inline-block`, `flex`, `inline-flex`, `table`, `table-row`, `table-cell` |
| `position`       | `static`, `absolute`             |
| `float`          | `left`, `right`, `none`          |
| `clear`          | `left`, `right`, `both`, `none`  |
| `overflow`       | `hidden`, `scroll`, `visible`    |
| `visibility`     | `visible`, `hidden`              |
| `opacity`        | `0.0` a `1.0`                    |

### Dimensões

| Propriedade  | Unidades                |
|--------------|-------------------------|
| `width`      | `px`, `%`, `auto`, `fit-content` |
| `height`     | `px`, `%`, `auto`, `fit-content` |
| `min-width`  | `px`, `%`               |
| `min-height` | `px`, `%`               |
| `max-width`  | `px`, `%`               |
| `max-height` | `px`, `%`               |

### Box Model

| Propriedade       | Unidades    |
|-------------------|-------------|
| `margin`          | `px`, `auto`|
| `margin-top/right/bottom/left` | `px`, `auto` |
| `padding`         | `px`        |
| `padding-top/right/bottom/left` | `px` |
| `border`          | `<width> <color>` (ex: `1 #ff0000`) |
| `border-top/right/bottom/left` | `<width> <color>` |

### Cores e Fundo

| Propriedade          | Formato                              |
|----------------------|--------------------------------------|
| `color`              | `#RRGGBB`, `#RRGGBBAA`, `rgb(r,g,b)`, `rgba(r,g,b,a)` |
| `background-color`   | Mesmo formato de `color`             |
| `border-color`       | Mesmo formato de `color`             |

### Tipografia

| Propriedade      | Valores                          |
|------------------|----------------------------------|
| `font-family`    | Nome da fonte                    |
| `font-size`      | `px`                             |
| `font-weight`    | `normal`, `bold`                 |
| `text-align`     | `left`, `center`, `right`        |
| `line-height`    | `px`                             |

### Flexbox

| Propriedade          | Valores                                          |
|----------------------|--------------------------------------------------|
| `flex-direction`     | `row`, `row-reverse`, `column`, `column-reverse` |
| `flex-wrap`          | `nowrap`, `wrap`, `wrap-reverse`                 |
| `justify-content`    | `flex-start`, `flex-end`, `center`, `space-between`, `space-around`, `space-evenly` |
| `align-items`        | `stretch`, `flex-start`, `flex-end`, `center`, `baseline` |
| `align-content`      | `stretch`, `flex-start`, `flex-end`, `center`, `space-between`, `space-around`, `space-evenly` |
| `align-self`         | `auto`, `stretch`, `flex-start`, `flex-end`, `center`, `baseline` |
| `flex-grow`          | Número (float)                   |
| `flex-shrink`        | Número (float)                   |
| `flex-basis`         | `auto`, `px`, `%`                |
| `order`              | Inteiro                           |
| `gap`                | `px`                              |
| `row-gap`            | `px`                              |
| `column-gap`         | `px`                              |

### Pseudo-classes

| Seletor              | Descrição                |
|----------------------|--------------------------|
| `:hover`             | Mouse sobre o widget     |
| `:focus`             | Widget com foco          |
| `:checked`           | Checkbox/Radio marcado   |
| `:disabled`          | Widget desabilitado      |
| `:pressed`           | Widget pressionado       |
| `:first-child`       | Primeiro filho           |
| `:last-child`        | Último filho             |
| `:nth-child(even)`   | Filhos pares             |
| `:nth-child(odd)`    | Filhos ímpares           |

---

## Data Binding

Atributos com prefixo `*` criam bindings reativos entre Lua e widgets.

### *text — Lua → Widget (unidirecional)

```html
<label *text="'Olá, ' .. self.nome" />
<label *text="tostring(self.vida) .. ' / ' .. tostring(self.vidaMax)" />
```
A expressão é reavaliada quando o widget é atualizado.

### *value — Widget ↔ Lua (bidirecional)

```html
<input type="text" *value="self.playerName" />
```
- Inicializa o input com `self.playerName`
- Quando o usuário digita, atualiza `self.playerName` automaticamente

### *checked — Widget ↔ Lua (bidirecional)

```html
<input type="checkbox" *checked="self.autoLoot" />
```

### *if — Condicional (remove do layout)

```html
<div *if="self.mostrarDebug">
  Debug info aqui...
</div>
```
Quando `false`, o elemento é removido do layout (não ocupa espaço).

### *visible — Visibilidade (mantém espaço)

```html
<div *visible="self.visivel">
  Este elemento sempre ocupa espaço
</div>
```
Quando `false`, fica invisível via `opacity: 0` mas mantém o espaço.

### *for — Loop reativo

```html
<div *for="local item in self.itens; local i = index">
  <label>{{i + 1}}. {{item}}</label>
</div>
```
- Renderiza um item para cada elemento da tabela
- Reativo: adiciona/remove widgets quando a tabela muda
- `index` é zero-based

### *outfit — Renderiza criatura

```html
<creature *outfit="self.meuOutfit" />
```
Onde `self.meuOutfit` é uma tabela `{ lookType = 128, lookHead = 0, ... }`.

---

## Diretivas de Fluxo

### Template Expressions `{{ }}`

Interpolação de expressões Lua no texto:

```html
<label>Olá {{self.nome}}, você tem {{self.idade}} anos</label>
<label>HP: {{self.vida}}/{{self.vidaMax}}</label>
```

---

## Eventos

### Atributos de Evento

| Atributo          | Dispara quando...                  |
|-------------------|------------------------------------|
| `onclick`         | Clique no widget                   |
| `ondoubleclick`   | Duplo clique                       |
| `onmousepress`    | Botão do mouse pressionado         |
| `onmouserelease`  | Botão do mouse solto               |
| `onmousemove`     | Mouse move sobre o widget          |
| `onmousewheel`    | Scroll do mouse                    |
| `onhover`         | Mouse entra/sai do widget          |
| `onfocus`         | Widget ganha foco                  |
| `onkeydown`       | Tecla pressionada                  |
| `onkeyup`         | Tecla solta                        |
| `onkeytext`       | Texto digitado                     |
| `onchange`        | Valor alterado (inputs)            |
| `ondragenter`     | Drag entra no widget               |
| `ondragleave`     | Drag sai do widget                 |
| `ondrop`          | Item solto no widget               |
| `onresize`        | Widget redimensionado              |
| `ondestroy`       | Widget destruído                   |

### Sintaxe de Eventos

```html
<button onclick="self:incrementar()">+1</button>
<button onclick="self.contador = self.contador + 1">+1</button>
<button onclick="self:removerItem(index)">X</button>
```

Dentro do handler, `self` é o controller do módulo. Para `*for`, `index` é o índice zero-based do item.

### Event Object

Dentro de eventos, a variável `event` contém:
```lua
event = {
    name = "onClick",     -- nome do evento
    target = widget,      -- widget alvo
    -- campos específicos por tipo de widget
}
```

---

## API Lua

### Controller

```lua
-- Criar controller
MeuModulo = Controller:new()

-- Ciclo de vida (sobrescreva)
function MeuModulo:onInit() end
function MeuModulo:onTerminate() end
function MeuModulo:onGameStart() end
function MeuModulo:onGameEnd() end

-- Carregar HTML
widget = self:loadHtml("arquivo.html")

-- Descarregar (destrói todos os widgets)
self:unloadHtml()

-- Criar widget de string HTML
self:createWidgetFromHTML("<label>Oi</label>", parent)

-- Buscar widgets por seletor CSS
widget = self:findWidget(".minha-classe")
widgets = self:findWidgets("button")

-- Atualizar loops *for após modificar tabelas
self:refreshFor()
```

### g_html (C++ singleton)

```lua
-- Carregar HTML de arquivo
local htmlId = g_html.load("meu_modulo", "janela.html", parentWidget)

-- Obter widget raiz
local root = g_html.getRootWidget(htmlId)

-- Criar widget de string HTML
local w = g_html.createWidgetFromHTML("<label>Oi</label>", parent, htmlId)

-- Adicionar stylesheet CSS global
g_html.addGlobalStyle("/data/styles/meu.css")

-- Destruir árvore HTML
g_html.destroy(htmlId)
```

### UIWidget (métodos HTML)

```lua
-- Query CSS selectors
widget = meuWidget:querySelector(".botao")
lista = meuWidget:querySelectorAll("label")

-- Manipular conteúdo HTML
meuWidget:append("<button>Novo</button>")
meuWidget:prepend("<label>Título</label>")
meuWidget:insert(2, "<div>Inserido</div>")
meuWidget:html("<div>Substitui tudo</div>")

-- Remover por seletor CSS
count = meuWidget:remove(".item-removivel")
```

---

## API C++

### HtmlManager (singleton `g_html`)

```cpp
// Carregar HTML
uint32_t id = g_html.load("moduleName", "path.html", parentWidget);

// Criar widget de string
auto widget = g_html.createWidgetFromHTML("<label>Oi</label>", parent, htmlId);

// Adicionar CSS global
g_html.addGlobalStyle("/data/styles/custom.css");

// Obter widget raiz
auto root = g_html.getRootWidget(htmlId);

// Destruir
g_html.destroy(htmlId);
g_html.terminate(); // limpa tudo
```

### UIWidget — Manipulação HTML

```cpp
// Query selectors
auto widget = parent->querySelector(".classe");
auto list = parent->querySelectorAll("button");

// Manipular DOM
parent->append("<div>Novo</div>");
parent->prepend("<span>Primeiro</span>");
parent->insert(2, "<label>Posição 2</label>");
parent->html("<div>Substitui conteúdo</div>");
size_t removed = parent->remove(".item");
```

---

## Exemplos Práticos

### Exemplo 1: Janela de status com contador

**janela.html:**
```html
<style>
  .panel {
    background-color: rgb(30, 30, 60);
    width: 280px;
    height: auto;
    text-align: center;
    padding: 10;
    border: 1 #444488;
  }
  .panel:hover {
    background-color: rgb(50, 50, 90);
  }
  button {
    display: block;
    margin-top: 5;
    width: fit-content;
  }
</style>

<script type="text">
  self.contador = 0
  self.ativo    = true
</script>

<html>
<div class="panel" placement="center">
  <label *text="'Contador: ' .. tostring(self.contador)" />
  <button onclick="self:incrementar()">+1</button>
  <button onclick="self.contador = self.contador - 1">-1</button>
  <div *if="self.ativo">
    <label>Sistema ativo ✓</label>
  </div>
  <button onclick="self:close()">Fechar</button>
</div>
</html>
```

**modulo.lua:**
```lua
MeuModulo = Controller:new()

function MeuModulo:onInit()
    self:loadHtml('janela.html')
end

function MeuModulo:incrementar()
    self.contador = self.contador + 1
end

function MeuModulo:close()
    self:unloadHtml()
end
```

### Exemplo 2: Lista reativa com *for

```html
<style>
  .item-row {
    display: block;
    margin: 2;
    padding: 4;
    background-color: rgb(40, 40, 80);
  }
</style>

<script type="text">
  self.itens = {'Espada', 'Escudo', 'Poção'}
</script>

<html>
<div placement="center">
  <div *for="local item in self.itens; local i = index">
    <div class="item-row">
      <label>{{i + 1}}. {{item}}</label>
      <button onclick="self:removerItem(index)">X</button>
    </div>
  </div>
  <button onclick="self:adicionarItem()">+ Item</button>
</div>
</html>
```

```lua
function MeuModulo:adicionarItem()
    table.insert(self.itens, 'Novo Item')
    self:refreshFor()  -- ATUALIZA o loop *for
end

function MeuModulo:removerItem(index)
    table.remove(self.itens, index + 1)  -- index é zero-based
    self:refreshFor()
end
```

### Exemplo 3: Tabela HTML

```html
<style>
  table {
    margin-top: 8;
    width: fit-content;
  }
  th {
    background-color: rgb(40, 40, 100);
    padding: 4;
  }
  td {
    padding: 4;
    border-bottom: 1 #333366;
  }
</style>

<table>
  <thead>
    <tr><th>Item</th><th>Qtd</th></tr>
  </thead>
  <tbody>
    <tr><td>Espada</td><td>1</td></tr>
    <tr><td>Poção</td><td>5</td></tr>
  </tbody>
</table>
```

### Exemplo 4: CSS customizado para widgets nativos

**data/styles/custom.css:**
```css
/* Todos os botões */
Button {
  margin: 5;
}

/* Painéis com padding */
QtPanel {
  padding: 10;
}

/* Input boxes */
TextEdit {
  padding: 4;
  padding-bottom: 1;
}

/* Minipanel como container relativo */
minipanel {
  position: relative;
}
```

### Exemplo 5: Usando Controller e findWidget

```lua
MeuModulo = Controller:new()

function MeuModulo:onInit()
    self:loadHtml('form.html')
    
    -- Buscar widget específico
    local btn = self:findWidget('#btn-salvar')
    if btn then
        btn:setEnabled(false)
    end
    
    -- Buscar todos os labels
    local labels = self:findWidgets('label')
    for _, lbl in ipairs(labels) do
        lbl:setColor('white')
    end
    
    -- Adicionar widget dinamicamente
    local container = self:findWidget('.lista')
    if container then
        container:append('<label>Item novo</label>')
    end
end
```

### Exemplo 6: Layout Flexbox

```html
<style>
  .flex-row {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    gap: 10;
    padding: 10;
  }
  .flex-col {
    display: flex;
    flex-direction: column;
    gap: 5;
  }
</style>

<div class="flex-row">
  <button>Botão 1</button>
  <button>Botão 2</button>
  <button>Botão 3</button>
</div>

<div class="flex-col">
  <label>Item A</label>
  <label>Item B</label>
  <label>Item C</label>
</div>
```

---

## CSS Selectors para Widgets Nativos

Você pode estilizar widgets OTClient pelo nome da classe em `custom.css`:

```css
/* Widget Button */
Button {
  padding: 5 10 5 10;
}

/* MiniWindow (janelas) */
MiniWindow {
  padding-top: 36;
}

/* HealthBar */
HealthBar {
  margin: 2;
}

/* Containers */
Container {
  padding: 4;
}

/* Scrollbar */
ScrollBar {
  width: 12;
}
```

Os nomes das classes correspondem ao `__class` definido nos arquivos `.otui` em `data/styles/`.

---

## Arquivos de Referência

| Arquivo | Descrição |
|---------|-----------|
| `modules/corelib/controller.lua` | Classe base para módulos HTML |
| `modules/corelib/html.lua` | Handlers de atributos, eventos, *for |
| `modules/test_html/` | Módulo de demonstração completo |
| `data/styles/html.otui` | Estilo base UIHTML |
| `data/styles/html.css` | CSS global |
| `data/styles/custom.css` | Sobrescritas de widgets nativos |
| `src/framework/html/` | Engine C++ |
| `src/framework/ui/uiwidgethtml.cpp` | Renderizador HTML |
| `src/framework/ui/uilayoutflexbox.cpp` | Layout engine |
