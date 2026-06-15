# Motor de Apresentação de Slides Vanilla JS

Este documento explica o funcionamento do sistema de apresentação de slides construído com HTML, Tailwind CSS e JavaScript puro (Vanilla JS). O motor é leve e baseia-se na manipulação de classes para exibir ou ocultar seções, além de escutar múltiplos eventos de interação do utilizador.

## 🏗️ Estrutura HTML Essencial

Para que o script de navegação funcione corretamente, o HTML deve possuir uma estrutura base ancorada em classes e IDs específicos.

### Componentes Obrigatórios
* **`.slide`**: Classe que define cada ecrã da apresentação.
* **`.active`**: Classe dinâmica controlada pelo JavaScript. O elemento que recebe esta classe é o slide atualmente visível.

### Componentes de Interface (Opcionais, mas recomendados)
* **`#progress-bar`**: Uma barra cujo estilo inline de largura (`width: X%`) é injetado dinamicamente.
* **`#counter`**: Elemento que recebe o número do slide atual.
* **`#total-slides`**: Elemento que recebe a quantidade total de slides disponíveis.

### Exemplo de Esqueleto HTML:
```html
<div class="container-progresso">
    <div id="progress-bar"></div>
</div>

<span id="counter"></span> / <span id="total-slides"></span>

<div class="slide active"> Slide 1 </div>
<div class="slide"> Slide 2 </div>
```

---

## 🎨 Estilo dos Controlos e Barra de Progresso

A interface sobrepõe-se aos slides utilizando um posicionamento fixo (`fixed`) e um alto `z-index`, garantindo que os controlos e a barra de progresso estão sempre visíveis e acessíveis, independentemente do conteúdo do slide atual. A estilização é feita primariamente com classes utilitárias do Tailwind CSS.

### 1. Barra de Progresso (`#progress-bar`)

A barra de progresso é composta por dois elementos: um *container* de fundo e a barra de preenchimento real.

* **Container (Fundo):**
  Fica ancorado no topo absoluto do ecrã ocupando 100% da largura.
  *Classes Tailwind:* `fixed top-0 left-0 w-full h-1.5 z-50 bg-slate-900`

* **Barra Dinâmica (`#progress-bar`):**
  Ocupa a altura total do seu container e possui um gradiente de cor. Começa com largura zero (`w-0`).
  *Classes Tailwind:* `h-full bg-gradient-to-r from-brand-500 to-accent-400 w-0`
  * **Animação (CSS Customizado):** Para que a barra não "salte" instantaneamente de um valor para o outro, é aplicada uma transição suave diretamente no CSS puro, utilizando uma curva de Bézier para um efeito mais natural:
    ```css
    #progress-bar {
        transition: width 0.4s cubic-bezier(0.4, 0, 0.2, 1);
    }
    ```

### 2. Contadores e Navegação (Helpers)

Os elementos de ajuda visual ficam ancorados na parte inferior do ecrã (`bottom-6`), em lados opostos.

* **Contador de Slides (Canto Inferior Direito):**
  Exibe a página atual em relação ao total.
  *Classes Tailwind:* `fixed bottom-6 right-8 z-50 font-display font-bold text-slate-500 tracking-widest text-lg`
  * O número atual (`#counter`) possui uma cor mais clara (`text-slate-300`) para se destacar do texto separador e do número total.

* **Dica de Navegação (Canto Inferior Esquerdo):**
  Uma instrução visual em texto pequeno para o utilizador, indicando como navegar (Teclado ou Clique).
  *Classes Tailwind:* `fixed bottom-6 left-8 z-50 text-slate-500 text-sm flex items-center gap-3`
  * **Estilo das Teclas (`<kbd>`):** Para simular a aparência de teclas físicas reais na interface, as tags `<kbd>` recebem formatação específica com fundos escuros, bordas e cantos arredondados: `bg-slate-800 px-2 py-1 rounded text-slate-300 border border-slate-700`.

---

## ⚙️ Como o JavaScript Funciona

Toda a lógica reside dentro de um listener `DOMContentLoaded` para garantir que o script apenas tenta manipular elementos que já foram desenhados no navegador.

### Variáveis de Estado
O sistema mantém o estado através de três variáveis principais:
1.  `slides`: Uma `NodeList` que captura todos os elementos contendo a classe `.slide`.
2.  `currentSlideIndex`: Um número inteiro (iniciado em `0`) que serve como ponteiro para o slide atual.
3.  `totalSlides`: O comprimento do array/NodeList de slides.

### A Função Central: `updatePresentation()`
Esta é a função responsável por renderizar o estado atual da apresentação na tela. Sempre que o utilizador navega, esta função é chamada. Ela executa 4 passos sequenciais:
1.  Itera sobre todos os `.slide` e remove a classe `.active` de todos eles.
2.  Adiciona a classe `.active` especificamente no slide cujo índice corresponde ao `currentSlideIndex`.
3.  Atualiza o texto do elemento `#counter` para `currentSlideIndex + 1` (para leitura humana).
4.  Calcula a percentagem exata `((índice + 1) / total) * 100` e injeta no CSS via `style.width` do `#progress-bar`.

### Funções de Navegação
* **`nextSlide()`**: Verifica se não estamos no último slide (`currentSlideIndex < totalSlides - 1`). Se for verdade, soma +1 ao índice e pede para atualizar a tela.
* **`prevSlide()`**: Verifica se não estamos no primeiro slide (`currentSlideIndex > 0`). Se for verdade, subtrai -1 ao índice e pede para atualizar a tela.

---

## 🎮 Interações e Eventos (Controlos)

O script foi concebido para suportar 3 métodos de navegação nativos:

### 1. Teclado (`keydown`)
Ouve as teclas premidas em qualquer parte da página.
* **Avançar**: Tecla <kbd>Espaço</kbd>, <kbd>Enter</kbd> ou <kbd>Seta para a Direita</kbd>. (Também previne o comportamento padrão destas teclas para evitar o scroll da página).
* **Recuar**: Tecla <kbd>Seta para a Esquerda</kbd>.

### 2. Clique do Rato (`click`)
Qualquer clique no ecrã invoca a função `nextSlide()`, com duas salvaguardas (tratamento de exceções) para não atrapalhar a usabilidade normal:
* Não avança se o utilizador estiver a selecionar texto na tela (`window.getSelection().toString().length === 0`).
* Não avança se o alvo do clique for um link (`<a>`) ou um botão (`<button>`).

### 3. Dispositivos Móveis / Swipe (`touchstart` & `touchend`)
Mede a distância do movimento do dedo no eixo horizontal da tela.
1.  No início do toque (`touchstart`), regista o ponto de partida `X` (`touchStartX`).
2.  Ao levantar o dedo (`touchend`), regista o ponto final `X` (`touchEndX`).
3.  Compara os dois pontos com uma margem de segurança de `50px` para evitar toques acidentais:
    * Se o ponto final for muito à esquerda do inicial (Arrastar para a esquerda), dispara `nextSlide()`.
    * Se o ponto final for muito à direita do inicial (Arrastar para a direita), dispara `prevSlide()`.
