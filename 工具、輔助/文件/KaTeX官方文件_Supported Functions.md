---
aliases:
date:
update:
author:
language:
sourceurl: https://katex.org/docs/supported
tags:
  - KaTeX
---

# 網站

- [官方網站](https://katex.org/)
- [Supported Functions](https://katex.org/docs/supported)
- [GitHub](https://github.com/KaTeX/KaTeX)

---

This is a list of TeX functions supported by KaTeX. It is sorted into logical groups.
這是 KaTeX 支援的 TeX 函數列表。它按邏輯組排序。

There is a similar [Support Table](https://katex.org/docs/support_table), sorted alphabetically, that lists both supported and un-supported functions.
有一個類似的 [支援表](https://katex.org/docs/support_table) ，按字母順序排列，列出了支援和不支援的功能。

# Accents ＊ 變音符號

|                               |                                                       |                                                 |
| ----------------------------- | ----------------------------------------------------- | ----------------------------------------------- |
| $a'$ `a'`                     | $\tilde{a}$ `\tilde{a}`                               | $\mathring{g}$ `\mathring{g}`                   |
| $a''$ `a''`                   | $\widetilde{ac}$ `\widetilde{ac}`                     | $\overgroup{AB}$ `\overgroup{AB}`               |
| $a^{\prime}$ `a^{\prime}`     | $\utilde{AB}$ `\utilde{AB}`                           | $\undergroup{AB}$ `\undergroup{AB}`             |
| $\acute{a}$ `\acute{a}`       | $\vec{F}$ `\vec{F}`                                   | $\Overrightarrow{AB}$ `\Overrightarrow{AB}`     |
| $\bar{y}$ `\bar{y}`           | $\overleftarrow{AB}$ `\overleftarrow{AB}`             | $\overrightarrow{AB}$ `\overrightarrow{AB}`     |
| $\breve{a}$ `\breve{a}`       | $\underleftarrow{AB}$ `\underleftarrow{AB}`           | $\underrightarrow{AB}$ `\underrightarrow{AB}`   |
| $\check{a}$ `\check{a}`       | $\overleftharpoon{ac}$ `\overleftharpoon{ac}`         | $\overrightharpoon{ac}$ `\overrightharpoon{ac}` |
| $\dot{a}$ `\dot{a}`           | $\overleftrightarrow{AB}$ `\overleftrightarrow{AB}`   | $\overbrace{AB}$ `\overbrace{AB}`               |
| $\ddot{a}$ `\ddot{a}`         | $\underleftrightarrow{AB}$ `\underleftrightarrow{AB}` | $\underbrace{AB}$ `\underbrace{AB}`             |
| $\grave{a}$ `\grave{a}`       | $\overline{AB}$ `\overline{AB}`                       | $\overlinesegment{AB}$ `\overlinesegment{AB}`   |
| $\hat{\theta}$ `\hat{\theta}` | $\underline{AB}$ `\underline{AB}`                     | $\underlinesegment{AB}$ `\underlinesegment{AB}` |
| $\widehat{ac}$ `\widehat{ac}` | $\widecheck{ac}$ `\widecheck{ac}`                     | $\underbar{X}$ `\underbar{X}`                   |

## Accent functions inside \text{…} ＊ 格式的强调函数

|                 |                 |                 |                 |
| --------------- | --------------- | --------------- | --------------- |
| $\'{a}$ `\'{a}` | $\~{a}$ `\~{a}` | $\.{a}$ `\.{a}` | $\H{a}$ `\H{a}` |
| $\`{a}$ `\`{a}` | $\={a}$ `\={a}` | $\"{a}$ `\"{a}` | $\v{a}$ `\v{a}` |
| $\^{a}$ `\^{a}` | $\u{a}$ `\u{a}` | $\r{a}$ `\r{a}` |

See also [letters and unicode](https://katex.org/docs/supported#letters-and-unicode).
另請參閱 [字母和 unicode](https://katex.org/docs/supported#letters-and-unicode) 。

---

# Delimiters ＊ 分隔符號

|                                 |                                     |                   |                                                     |                                     |
| ------------------------------- | ----------------------------------- | ----------------- | --------------------------------------------------- | ----------------------------------- |
| $( )$ `( )`                     | $\lparen \rparen$ `\lparen \rparen` | $⌈ ⌉$ `⌈ ⌉`       | $\lceil$ $\rceil$ `\lceil` `\rceil`                 | $\uparrow$ `\uparrow`               |
| $[ ]$ `[ ]`                     | $\lbrack \rbrack$ `\lbrack \rbrack` | $⌊ ⌋$ `⌊ ⌋`       | $\lfloor \rfloor$ `\lfloor \rfloor`                 | $\downarrow$ `\downarrow`           |
| $\{ \}$ `\{ \}`                 | $\lbrace \rbrace$ `\lbrace \rbrace` | $⎰⎱$ `⎰⎱`         | $\lmoustache \rmoustache$ `\lmoustache \rmoustache` | $\updownarrow$ `\updownarrow`       |
| $⟨ ⟩$ `⟨ ⟩`                     | $\langle \rangle$ `\langle \rangle` | $⟮ ⟯$ `⟮ ⟯`       | $\lgroup \rgroup$ `\lgroup \rgroup`                 | $\Uparrow$ `\Uparrow`               |
| $\|$ `\|`                       | $\vert$ `\vert`                     | $┌ ┐$ `┌ ┐`       | $\ulcorner \urcorner$ `\ulcorner \urcorner`         | $\Downarrow$ `\Downarrow`           |
| $\|\|$ `\|\|`                   | $\Vert$ `\Vert`                     | $└ ┘$ `└ ┘`       | $\llcorner \lrcorner$ `\llcorner \lrcorner`         | $\Updownarrow$ `\Updownarrow`       |
| $\lvert \rvert$ `\lvert \rvert` | $\lVert \rVert$ `\lVert \rVert`     | $\left.$ `\left.` | $\right.$ `\right.`                                 | $\backslash$ `\backslash`           |
| $\lang \rang$ `\lang \rang`     | $\lt \gt$ `\lt \gt`                 | $⟦ ⟧$ `⟦ ⟧`       | $\llbracket \rrbracket$ `\llbracket \rrbracket`     | $\lBrace \rBrace$ `\lBrace \rBrace` |

## Delimiter Sizing ＊ 分隔符號尺寸

$\left(\LARGE{AB}\right)$ `\left(\LARGE{AB}\right)`
$( \big( \Big( \bigg( \Bigg($ `( \big( \Big( \bigg( \Bigg(`

|           |         |          |          |          |
| --------- | ------- | -------- | -------- | -------- |
| `\left`   | `\big`  | `\bigl`  | `\bigm`  | `\bigr`  |
| `\middle` | `\Big`  | `\Bigl`  | `\Bigm`  | `\Bigr`  |
| `\right`  | `\bigg` | `\biggl` | `\biggm` | `\biggr` |
|           | `\Bigg` | `\Biggl` | `\Biggm` | `\Biggr` |

---

# Environments ＊ 環境

註：這裡都是用 `$$`。

|                                                                      |                                                                                        |                                                                                                                   |                                                                                                                                                   |
| -------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| $$\begin{matrix} a & b \\ c & d \end{matrix}$$                       | `\begin{matrix}`<br/>` a & b \\`<br/>` c & d`<br/>`\end{matrix}`                       | $$\begin{array}{cc} a & b \\ c & d \end{array}$$                                                                  | `\begin{array}{cc}`<br/>` a & b \\`<br/>` c & d`<br/>`\end{array}`                                                                                |
| $$\begin{bmatrix} a & b \\ c & d \end{bmatrix}$$                     | `\begin{bmatrix}`<br/>` a & b \\`<br/>` c & d`<br/>`\end{bmatrix}`                     | $$\begin{Bmatrix} a & b \\ c & d \end{Bmatrix}$$                                                                  | `\begin{Bmatrix}`<br/>` a & b \\`<br/>` c & d`<br/>`\end{Bmatrix}`                                                                                |
| $$\begin{vmatrix} a & b \\ c & d \end{vmatrix}$$                     | `\begin{vmatrix}`<br/>` a & b \\`<br/>` c & d`<br/>`\end{vmatrix}`                     | $$\begin{Vmatrix} a & b \\ c & d \end{Vmatrix}$$                                                                  | `\begin{Vmatrix}`<br/>` a & b \\`<br/>` c & d`<br/>`\end{Vmatrix}`                                                                                |
| $$\begin{pmatrix} a & b \\ c & d \end{pmatrix}$$                     | `\begin{pmatrix}`<br/>` a & b \\`<br/>` c & d`<br/>`\end{pmatrix}`                     | $$\def\arraystretch{1.5} \begin{array}{c:c:c} a & b & c \\ \hline d & e & f \\ \hdashline g & h & i \end{array}$$ | `\def\arraystretch{1.5}`<br/>` \begin{array}{c:c:c}`<br/>` a & b & c \\`<br/>` \hline d & e & f \\`<br/>` \hdashline g & h & i`<br/>`\end{array}` |
| $$x = \begin{cases} a &\text{if } b \\ c &\text{if } d \end{cases}$$ | `x = \begin{cases}`<br/>` a &\text{if } b \\`<br/>` c &\text{if } d`<br/>`\end{cases}` | $$\begin{rcases} a &\text{if } b \\ c &\text{if } d \end{rcases}⇒…$$                                              | `\begin{rcases}`<br/>` a &\text{if } b \\`<br/>` c &\text{if } d`<br/>`\end{rcases}⇒…`                                                            |
| $$\begin{smallmatrix} a & b \\ c & d \end{smallmatrix}$$             | `\begin{smallmatrix}`<br/>` a & b \\`<br/>` c & d`<br/>`\end{smallmatrix}`             | $$\sum_{\begin{subarray}{l} i\in\Lambda \\ 0<j<n \end{subarray}}$$                                                | `\sum_{`<br/>`\begin{subarray}{l}`<br/>` i\in\Lambda \\`<br/>` 0<j<n`<br/>`\end{subarray}}`                                                       |

The auto-render extension will render the following environments even if they are not inside math delimiters such as `$$…$$`. They are display-mode only.
自動渲染擴充功能將渲染以下環境，即使它們不在數學分隔符號內，例如 `$$…$$`。 它們僅用於顯示模式。

$$\begin{equation} \begin{split} a &=b+c \\ &=e+f \end{split}\end{equation}$$

`\begin{equation}`<br/>`\begin{split}`<br/>` a &=b+c \\`<br/>` &=e+f`<br/>`\end{split}`<br/>`\end{equation}`

$$\begin{align} a&=b+c \\ d+e&=f \end{align}$$ | `\begin{align}`<br/>` a&=b+c \\`<br/>` d+e&=f`<br/>`\end{align}`

$$\begin{gather} a=b \\ e=b+c \end{gather}$$

`\begin{gather}`<br/>` a=b \\`<br/>` e=b+c`<br/>`\end{gather}`

$$\begin{alignat}{2} 10&x+&3&y=2 \\ 3&x+&13&y=4 \end{alignat}$$

`\begin{alignat}{2}`<br/>` 10&x+&3&y=2 \\`<br/>`3&x+&13&y=4`<br/>`\end{alignat}`

$$\begin{CD} A @>a>> B \\ @VbVV @AAcA \\ C @= D \end{CD}$$

`\begin{CD}`<br/>` A @>a>> B \\`<br/>` @VbVV @AAcA \\`<br/>` C @= D`<br/>`\end{CD}`

## Other KaTeX Environments ＊ 其他 KaTex 環境

| Environments                                                         | How they differ from those shown above                                                                                                                             |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `darray`, `dcases`, `drcases`                                        | … apply `displaystyle`                                                                                                                                             |
| `matrix*`, `pmatrix*`, `bmatrix*`, `Bmatrix*`, `vmatrix*`,`Vmatrix*` | … take an optional argument to set column alignment, as in `\begin{matrix*}[r]`                                                                                    |
| `equation*`, `gather*`, `align*`, `alignat\*`                        | … have no automatic numbering. Alternatively, you can use `\nonumber` or `\notag` to omit the numbering for a specific row of the equation.                        |
| `gathered`, `aligned`, `alignedat`                                   | … do not need to be in display mode.<br/>… have no automatic numbering.<br/>… must be inside math delimiters in order to be rendered by the auto-render extension. |

Acceptable line separators include: `\\`, `\cr`, `\\[distance]`, and `\cr[distance]`. _Distance_ can be written with any of the [KaTeX units](https://katex.org/docs/supported#units).
可接受的行分隔符號包括： `\\` 、 `\cr` 、 `\\[distance]` 和 `\cr[distance]` 。 Distance 可以用任何 [KaTeX 單位](https://katex.org/docs/supported#units)_表示_ 。

The `{array}` environment supports `|` and `:` vertical separators.
`{array}` 環境支援 `|` 和 `:` 垂直分隔符號。

The `{array}` environment does not yet support `\cline` or `\multicolumn`.
`{array}` 環境尚不支援 `\cline` 或 `\multicolumn` 。

`\tag` can be applied to individual rows of top-level environments (`align`, `align*`, `alignat`, `alignat*`, `gather`, `gather*`).
`\tag` 可以應用於頂級環境（ `align` 、 `align*` 、 `alignat` 、 `alignat*` 、 `gather` 、 `gather*` ）的各個行。

---

# HTML

The following "raw HTML" features are potentially dangerous for untrusted inputs, so they are disabled by default, and attempting to use them produces the command names in red (which you can configure via the `errorColor` [option](https://katex.org/docs/options)). To fully trust your LaTeX input, you need to pass an option of `trust: true`; you can also enable just some of the commands or for just some URLs via the `trust` [option](https://katex.org/docs/options).
以下「原始 HTML」功能對於不受信任的輸入可能存在危險，因此預設為停用狀態，嘗試使用它們會導致命令名稱顯示為紅色（您可以透過 `errorColor` [選項](https://katex.org/docs/options) 進行配置）。要完全信任您的 LaTeX 輸入，您需要傳遞 `trust: true` 選項；您也可以透過 `trust` [選項](https://katex.org/docs/options) 僅啟用部分命令或部分 URL。

|                                                                     |                                                                                                                       |
| ------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| [KATE​X](https://katex.org/)                                        | `\href{https://katex.org/}{\KaTeX}`                                                                                   |
| https://katex.org/[https://katex.org/](https://katex.org/)          | `\url{https://katex.org/}`                                                                                            |
| ![KA logo](https://katex.org/img/khan-academy.png)                  | `\includegraphics[height=0.8em, totalheight=0.9em, width=0.9em, alt=KA logo]{https://katex.org/img/khan-academy.png}` |
| xx `…<span id="bar" class="enclosing">…x…</span>…`                  | `\htmlId{bar}{x}`                                                                                                     |
| xx `…<span class="enclosing foo">…x…</span>…`                       | `\htmlClass{foo}{x}`                                                                                                  |
| xx `…<span style="color: red;" class="enclosing">…x…</span>…`       | `\htmlStyle{color: red;}{x}`                                                                                          |
| xx `…<span data-foo="a" data-bar="b" class="enclosing">…x…</span>…` | `\htmlData{foo=a, bar=b}{x}`                                                                                          |

`\includegraphics` supports `height`, `width`, `totalheight`, and `alt` in its first argument. `height` is required.
`\includegraphics` 在第一個參數支援 `height` 、 `width` 、 `totalheight` 和 `alt` 。 `height` 是必需的。

HTML extension (`\html`-prefixed) commands are non-standard, so loosening `strict` option for `htmlExtension` is required.
HTML 擴充（ `\html` 前綴）指令是非標準的，因此需要放寬 `htmlExtension` 的 `strict` 選項。

# Letters and Unicode ＊ 字母和 Unicode

## Greek Letters ＊ 希臘字母

Direct Input :
ABΓΔEZHΘIKΛMNΞOΠPΣTΥΦXΨΩABΓΔEZHΘIKΛMNΞOΠPΣTΥΦXΨΩ αβγδϵζηθικλμνξoπρστυϕχψωεϑϖϱςφϝαβγδϵζηθικλμνξoπρστυϕχψωεϑϖϱςφϝ

|                             |                         |                         |                             |
| --------------------------- | ----------------------- | ----------------------- | --------------------------- |
| $\Alpha$ `\Alpha`           | $\Beta$ `\Beta`         | $\Gamma$ `\Gamma`       | $\Delta$ `\Delta`           |
| $\Epsilon$ `\Epsilon`       | $\Zeta$ `\Zeta`         | $\Eta$ `\Eta`           | $\Theta$ `\Theta`           |
| $\Iota$ `\Iota`             | $\Kappa$ `\Kappa`       | $\Lambda$ `\Lambda`     | $\Mu$ `\Mu`                 |
| $\Nu$ `\Nu`                 | $\Xi$ `\Xi`             | $\Omicron$ `\Omicron`   | $\Pi$ `\Pi`                 |
| $\Rho$ `\Rho`               | $\Sigma$ `\Sigma`       | $\Tau$ `\Tau`           | $\Upsilon$ `\Upsilon`       |
| $\Phi$ `\Phi`               | $\Chi$ `\Chi`           | $\Psi$ `\Psi`           | $\Omega$ `\Omega`           |
| $\varGamma$ `\varGamma`     | $\varDelta$ `\varDelta` | $\varTheta$ `\varTheta` | $\varLambda$ `\varLambda`   |
| $\varXi$ `\varXi`           | $\varPi$ `\varPi`       | $\varSigma$ `\varSigma` | $\varUpsilon$ `\varUpsilon` |
| $\varPhi$ `\varPhi`         | $\varPsi$ `\varPsi`     | $\varOmega$ `\varOmega` |                             |
| $\alpha$ `\alpha`           | $\beta$ `\beta`         | $\gamma$ `\gamma`       | $\delta$ `\delta`           |
| $\epsilon$ `\epsilon`       | $\zeta$ `\zeta`         | $\eta$ `\eta`           | $\theta$ `\theta`           |
| $\iota$ `\iota`             | $\kappa$ `\kappa`       | $\lambda$ `\lambda`     | $\mu$ `\mu`                 |
| $\nu$ `\nu`                 | $\xi$ `\xi`             | $\omicron$ `\omicron`   | $\pi$ `\pi`                 |
| $\rho$ `\rho`               | $\sigma$ `\sigma`       | $\tau$ `\tau`           | $\upsilon$ `\upsilon`       |
| $\phi$ `\phi`               | $\chi$ `\chi`           | $\psi$ `\psi`           | $\omega$ `\omega`           |
| $\varepsilon$ `\varepsilon` | $\varkappa$ `\varkappa` | $\vartheta$ `\vartheta` | $\thetasym$ `\thetasym`     |
| $\varpi$ `\varpi`           | $\varrho$ `\varrho`     | $\varsigma$ `\varsigma` | $\varphi$ `\varphi`         |
| $\digamma$ `\digamma`       |                         |                         |                             |

## Other Letters ＊ 其他字母

|                       |                       |                       |                           |                           |
| --------------------- | --------------------- | --------------------- | ------------------------- | ------------------------- |
| $\imath$ `\imath`     | $\nabla$ `\nabla`     | $\Im$ `\Im`           | $\Reals$ `\Reals`         | $\text{\OE}$ `\text{\OE}` |
| $\jmath$ `\jmath`     | $\partial$ `\partial` | $\image$ `\image`     | $\wp$ `\wp`               | $\text{\o}$ `\text{\o}`   |
| $\aleph$ `\aleph`     | $\Game$ `\Game`       | $\Bbbk$ `\Bbbk`       | $\weierp$ `\weierp`       | $\text{\O}$ `\text{\O}`   |
| $\alef$ `\alef`       | $\Finv$ `\Finv`       | $\N$ `\N`             | $\Z$ `\Z`                 | $\text{\ss}$ `\text{\ss}` |
| $\alefsym$ `\alefsym` | $\cnums$ `\cnums`     | $\natnums$ `\natnums` | $\text{\aa}$ `\text{\aa}` | $\text{\i}$ `\text{\i}`   |
| $\beth$ `\beth`       | $\Complex$ `\Complex` | $\R$ `\R`             | $\text{\AA}$ `\text{\AA}` | $\text{\j}$ `\text{\j}`   |
| $\gimel$ `\gimel`     | $\ell$ `\ell`         | $\Re$ `\Re`           | $\text{\ae}$ `\text{\ae}` |
| $\daleth$ `\daleth`   | $\hbar$ `\hbar`       | $\real$ `\real`       | $\text{\AE}$ `\text{\AE}` |
| $\eth$ `\eth`         | $\hslash$ `\hslash`   | $\reals$ `\reals`     | $\text{\oe}$ `\text{\oe}` |

Direct Input :
∂∇ℑℲℵℶℷℸ⅁ℏð−∗∂∇ℑℲℵℶℷℸ⅁ℏð−∗ ÀÁÂÃÄÅÆÇÈÉÊËÌÍÎÏÐÑÒÓÔÕÖÙÚÛÜÝÞßàáâãäåçèéêëìíîïðñòóôöùúûüýþÿ ₊₋₌₍₎₀₁₂₃₄₅₆₇₈₉ₐₑₕᵢⱼₖₗₘₙₒₚᵣₛₜᵤᵥₓᵦᵧᵨᵩᵪ⁺⁻⁼⁽⁾⁰¹²³⁴⁵⁶⁷⁸⁹ᵃᵇᶜᵈᵉᵍʰⁱʲᵏˡᵐⁿᵒᵖʳˢᵗᵘʷˣʸᶻᵛᵝᵞᵟᵠᵡ

Math-mode Unicode (sub|super)script characters will render as if you had written regular characters in a subscript or superscript. For instance, `A²⁺³` will render the same as `A^{2+3}`.
數學模式下的 Unicode 上標或下標字元的渲染效果，會如同您在上標或下標中寫入了常規字元一樣。例如， `A²⁺³` 渲染效果與 `A^{2+3}` 相同。

## Unicode Mathematical Alphanumeric Symbols ＊ Unicode 數學字母數字符號

| Item        | Range                                  | Item              | Range                  |
| ----------- | -------------------------------------- | ----------------- | ---------------------- |
| Bold        | $\bold{𝐀-𝐙 \enspace 𝐚-𝐳 \enspace 𝟎-𝟗}$ | Double-struck     | $\Bbb{A-Z \enspace k}$ |
| Italic      | A-Z a-z                                | Sans serif        | A-Z a-z 0-9            |
| Bold Italic | A-Z a-z                                | Sans serif bold   | A-Z a-z 0-9            |
| Script      | A-Z                                    | Sans serif italic | A-Z a-z                |
| Fractur     | A-Z a-z                                | Monospace         | A-Z a-z 0-9            |

### Unicode

The letters listed above will render properly in any KaTeX rendering mode.
上面列出的字母將在任何 KaTeX 渲染模式下正確渲染。

In addition, Armenian, Brahmic, Georgian, Chinese, Japanese, and Korean glyphs are always accepted in text mode. However, these glyphs will be rendered from system fonts (not KaTeX-supplied fonts) so their typography may clash. You can provide rules for CSS classes `.latin_fallback`, `.cyrillic_fallback`, `.brahmic_fallback`, `.georgian_fallback`, `.cjk_fallback`, and `.hangul_fallback` to provide fallback fonts for these languages. Use of these glyphs may cause small vertical alignment issues: KaTeX has detailed metrics for listed symbols and most Latin, Greek, and Cyrillic letters, but other accepted glyphs are treated as if they are each as tall as the letter M in the current KaTeX font.
此外，亞美尼亞語、婆羅米語、格魯吉亞語、中文、日語和韓語的字形在文本模式下始終被接受。但是，這些字形將使用系統字體（而非 KaTeX 提供的字體）進行渲染，因此它們的排版可能會發生衝突。您可以為 CSS 類別 `.latin_fallback` 、 `.cyrillic_fallback` 、 `.brahmic_fallback` 、 `.georgian_fallback` 、 `.cjk_fallback` 和 `.hangul_fallback` 提供規則，為這些語言提供後備字體。使用這些字形可能會導致輕微的垂直對齊問題：KaTeX 對列出的符號以及大多數拉丁字母、希臘字母和西里爾字母都有詳細的度量，但其他可接受的字形在當前 KaTeX 字體中被視為與字母 M 一樣高。

If the KaTeX rendering mode is set to `strict: false` or `strict: "warn"` (default), then KaTeX will accept all Unicode letters in both text and math mode. All unrecognized characters will be treated as if they appeared in text mode, and are subject to the same issues of using system fonts and possibly using incorrect vertical alignment.
如果 KaTeX 渲染模式設定為 `strict: false` 或 `strict: "warn"` （預設），則 KaTeX 將在文字和數學模式下接受所有 Unicode 字母。所有無法識別的字元將被視為在文字模式下出現，並且會遇到與使用系統字體相同的問題，並且可能使用不正確的垂直對齊方式。

For Persian composite characters, a user-supplied [plug-in](https://github.com/HosseinAgha/persian-katex-plugin) is under development.
對於波斯語複合字符，用戶提供的 [插件](https://github.com/HosseinAgha/persian-katex-plugin) 正在開發中。

Any character can be written with the `\char` function and the Unicode code in hex. For example `\char"263a` will render as ☺☺.
任何字元都可以用 `\char` 函數和十六進位 Unicode 碼來表示。例如， `\char"263a` 將呈現為 ☺☺ 。

---

# Layout ＊ 佈局

## Annotation ＊ 註解

|                     |                     |                                    |                                    |
| ------------------- | ------------------- | ---------------------------------- | ---------------------------------- |
| $\cancel{5}$        | `\cancel{5}`        | $\overbrace{a+b+c}^{\text{note}}$  | `\overbrace{a+b+c}^{\text{note}}`  |
| $\bcancel{5}$       | `\bcancel{5}`       | $\underbrace{a+b+c}_{\text{note}}$ | `\underbrace{a+b+c}_{\text{note}}` |
| $\xcancel{ABC}$     | `\xcancel{ABC}`     | $\not=$                            | `\not=`                            |
| $\sout{abc}$        | `\sout{abc}`        | $\boxed{\pi=\frac c d}$            | `\boxed{\pi=\frac c d}`            |
| $a_{\angl n}$       | `a_{\angl n}`       | $a_\angln$                         | `a_\angln`                         |
| $\phase{-78^\circ}$ | `\phase{-78^\circ}` |

$$\tag{hi} x+y^{2x}$$

`\tag{hi} x+y^{2x}`

$$\tag*{hi} x+y^{2x}$$

`\tag*{hi} x+y^{2x}`

## Line Breaks ＊ 換行符號

KaTeX 0.10.0+ will insert automatic line breaks in inline math after relations or binary operators such as “=” or “+”. These can be suppressed by `\nobreak` or by placing math inside a pair of braces, as in `{F=ma}`. `\allowbreak` will allow automatic line breaks at locations other than relations or operators.
KaTeX 0.10.0+ 版本會在內聯數學公式中，在關係運算子或二元運算子（例如「=」或「+」）後插入自動換行符。您可以使用 `\nobreak` 或將數學公式放在括號內（例如 `{F=ma}` 來停用自動換行符。 `\allowbreak` 允許在關係運算子或運算子以外的位置自動換行。

Hard line breaks are `\\` and `\newline`.
硬換行符號是 `\\` 和 `\newline` 。

In display math, KaTeX does not insert automatic line breaks. It ignores display math hard line breaks when rendering option `strict: true`.
在顯示數學中，KaTeX 不會插入自動換行符號。當渲染選項 `strict: true` 時，它會忽略顯示數學的硬換行符。

## Vertical Layout ＊ 垂直佈局

註：有些須用 `$$`。

|        |        |                   |                   |                                                         |                                                       |
| ------ | ------ | ----------------- | ----------------- | ------------------------------------------------------- | ----------------------------------------------------- |
| $x_n$  | `x_n`  | $\stackrel{!}{=}$ | `\stackrel{!}{=}` | $a \atop b$                                             | `a \atop b`                                           |
| $e^x$  | `e^x`  | $\overset{!}{=}$  | `\overset{!}{=}`  | $$a\raisebox{0.25em}{$b$}c$$                            | `a\raisebox{0.25em}{$b$}c`                            |
| $_u^o$ | `_u^o` | $\underset{!}{=}$ | `\underset{!}{=}` | $$a+\left(\vcenter{\hbox{$\frac{\frac a b}c$}}\right)$$ | `a+\left(\vcenter{\hbox{$\frac{\frac a b}c$}}\right)` |
|        |        |                   |                   | $\sum_{\substack{0<i<m\\0<j<n}}$                        | `\sum_{\substack{0<i<m\\0<j<n}}`                      |

`\raisebox` and `\hbox` put their argument into text mode. To raise math, nest `$…$` delimiters inside the argument as shown above.
`\raisebox` 和 `\hbox` 將其參數置於文字模式。若要提高數學運算，請在參數內部嵌套 `$…$` 分隔符，如上所示。

`\vcenter` can be written without an `\hbox` if the `strict` rendering option is _false_. In that case, omit the nested `$…$` delimiters.
如果 `strict` 渲染選項為 _false_ ，則 `\vcenter` 可以不帶 `\hbox` 。在這種情況下，請省略巢狀的 `$…$` 分隔符號。

## Overlap and Spacing ＊ 重疊和間距

|                                          |                                                           |
| ---------------------------------------- | --------------------------------------------------------- |
| ${=}\mathllap{/\,}$ `${=}\mathllap{/\,}` | $\left(x^{\smash{2}}\right)$ `\left(x^{\smash{2}}\right)` |
| $\mathrlap{\,/}{=}$ `\mathrlap{\,/}{=}`  | $\sqrt{\smash[b]{y}}$ `\sqrt{\smash[b]{y}}`               |

$$\sum_{\mathclap{1\le i\le j\le n}} x_{ij}$$

`\sum_{\mathclap{1\le i\le j\le n}} x_{ij}`

KaTeX also supports `\llap`, `\rlap`, and `\clap`, but they will take only text, not math, as arguments.
KaTeX 也支援 `\llap` 、 `\rlap` 和 `\clap` ，但它們只接受文字（而不是數學）作為參數。

### Spacing ＊ 間距

| Produces           | Sample             | Function         | Produces                            | Sample             | Function         |
| ------------------ | ------------------ | ---------------- | ----------------------------------- | ------------------ | ---------------- |
| ³∕₁₈ em space      | █$\,$█             | `\,`             | distance 1ex                        | █$\kern{1ex}$█     | `\kern{1ex}`     |
| ³∕₁₈ em space      | █$\thinspace$█     | `\thinspace`     | distance 1em                        | █$\mkern{1em}$█    | `\mkern{1em}`    |
| ⁴∕₁₈ em space      | █$\>$█             | `\>`             | distance 18mu = 1em                 | █$\mskip{18mu}$█   | `\mskip{18mu}`   |
| ⁴∕₁₈ em space      | █$\:$█             | `\:`             | distance 1pc = 12 KaTeX pt          | █$\hskip{1pc}$█    | `\hskip{1pc}`    |
| ⁴∕₁₈ em space      | █$\medspace$█      | `\medspace`      | distance 10dd = 1238/1157​ KaTeX pt | █$\hspace{10dd}$█  | `\hspace{10dd}`  |
| ⁵∕₁₈ em space      | █$\;$█             | `\;`             | distance 1cc = 14856/1157 KaTeX pt  | █$\hspace*{1cc}$█  | `\hspace*{1cc}`  |
| ⁵∕₁₈ em space      | █$\thickspace$█    | `\thickspace`    | distance 1nd = 685/642 KaTeX pt     | █$\phantom{1nd}$█  | `\phantom{1nd}`  |
| ½ em space         | █$\enspace$█       | `\enspace`       | distance 1nc = 1370/107​ KaTeX pt   | █$\hphantom{1nd}$█ | `\hphantom{1nd}` |
| 1 em space         | █$\quad$█          | `\quad`          | distance 65536sp = 1 KaTeX pt       | █$\kern{65536sp}$█ | `\kern{65536sp}` |
| 2 em space         | █$\qquad$█         | `\qquad`         | v distance 1em                      | █$\vphantom{1em}$█ | `\vphantom{1em}` |
| – ³∕₁₈ em space    | █$\negthinspace$█  | `\negthinspace`  | distance 1in                        | █$\kern{1in}$█     | `\kern{1in}`     |
| – ³∕₁₈ em space    | █$\!$█             | `\!`             | distance 72bp                       | █$\kern{72bp}$█    | `\kern{72bp}`    |
| – ⁴∕₁₈ em space    | █$\negmedspace$█   | `\negmedspace`   | distance 72.27pt                    | █$\kern{72.27pt}$█ | `\kern{72.27pt}` |
| – ⁵∕₁₈ em space    | █$\negthickspace$█ | `\negthickspace` | distance 1mm                        | █$\kern{1mm}$█     | `\kern{1mm}`     |
| non-breaking space | █$~$█              | `~`              | distance 1cm                        | █$\kern{1cm}$█     | `\kern{1cm}`     |
| non-breaking space | █$\nobreakspace$█  | `\nobreakspace`  |

Notes:

`distance` will accept any of the [KaTeX units](https://katex.org/docs/supported#units).
`distance` 將接受任何 [KaTeX 單位](https://katex.org/docs/supported#units) 。

`\kern`, `\mkern`, `\mskip`, and `\hspace` accept unbraced distances, as in: `\kern1em`.
`\kern` 、 `\mkern` 、 `\mskip` 和 `\hspace` 接受無支撐距離，例如： `\kern1em` 。

`\mkern` and `\mskip` will not work in text mode and both will write a console warning for any unit except `mu`.
`\mkern` 和 `\mskip` 在文字模式下不起作用，並且兩者都會對 `mu` 以外的任何單位寫入控制台警告。

---

# Logic and Set Theory ＊ 邏輯與集合論

|                       |                                                               |                                         |                                   |
| --------------------- | ------------------------------------------------------------- | --------------------------------------- | --------------------------------- |
| $\forall$ `\forall`   | $\complement$ `\complement`                                   | $\therefore$ `\therefore`               | $\emptyset$ `\emptyset`           |
| $\exists$ `\exists`   | $\subset$ `\subset`                                           | $\because$ `\because`                   | $\empty$ `\empty`                 |
| $\exist$ `\exist`     | $\supset$ `\supset`                                           | $\mapsto$ `\mapsto`                     | $\varnothing$ `\varnothing`       |
| $\nexists$ `\nexists` | $\mid$ `\mid`                                                 | $\to$ `\to`                             | $\implies$ `\implies`             |
| $\in$ `\in`           | $\land$ `\land`                                               | $\gets$ `\gets`                         | $\impliedby$ `\impliedby`         |
| $\isin$ `\isin`       | $\lor$ `\lor`                                                 | $\leftrightarrow$ `\leftrightarrow`     | $\iff$ `\iff`                     |
| $\notin$ `\notin`     | $\ni$ `\ni`                                                   | $\notni$ `\notni`                       | $\neg$ `\neg`<br/>$\lnot$ `\lnot` |
|                       | $\Set{ x \mid x<\frac 1 2 }$<br/>`\Set{ x \mid x<\frac 1 2 }` | $\set{x\mid x<5}$<br/>`\set{x\mid x<5}` |
Direct Input :
∀∴∁∵∃∣∈∉∋⊂⊃∧∨↦→←↔¬∀∴∁∵∃∣∈∈/∋⊂⊃∧∨↦→←↔¬ ℂ ℍ ℕ ℙ ℚ ℝ

---

# Macros ＊ 巨集

|                                                   |                                                   |
| ------------------------------------------------- | ------------------------------------------------- |
| $\def\foo{x^2} \foo + \foo$                       | `\def\foo{x^2} \foo + \foo`                       |
| $\gdef\foo#1{#1^2} \foo{y} + \foo{y}$             | `\gdef\foo#1{#1^2} \foo{y} + \foo{y}`             |
| $\edef\macroname#1#2…{definition to be expanded}$ | `\edef\macroname#1#2…{definition to be expanded}` |
| $\xdef\macroname#1#2…{definition to be expanded}$ | `\xdef\macroname#1#2…{definition to be expanded}` |
| $\let\foo=\bar$                                   | `\let\foo=\bar`                                   |
| $\futurelet\foo\bar x$                            | `\futurelet\foo\bar x`                            |
| $\global\def\macroname#1#2…{definition}$          | `\global\def\macroname#1#2…{definition}`          |
| $\newcommand\macroname[numargs]{definition}$      | `\newcommand\macroname[numargs]{definition}`      |
| $\renewcommand\macroname[numargs]{definition}$    | `\renewcommand\macroname[numargs]{definition}`    |
| $\providecommand\macroname[numargs]{definition}$  | `\providecommand\macroname[numargs]{definition}`  |

Macros can also be defined in the KaTeX [rendering options](https://katex.org/docs/options).
巨集也可以在 KaTeX [渲染選項](https://katex.org/docs/options) 中定義。

Macros accept up to nine arguments: #1, #2, etc.
巨集最多接受九個參數：#1、#2 等。

Macros defined by `\gdef`, `\xdef`, `\global\def`, `\global\edef`, `\global\let`, and `\global\futurelet` will persist between math expressions. (Exception: macro persistence may be disabled. There are legitimate security reasons for that.)
由 `\gdef` 、 `\xdef` 、 `\global\def` 、 `\global\edef` 、 `\global\let` 和 `\global\futurelet` 定義的巨集將在數學表達式之間保留。 （例外：巨集保留功能可能被停用。這是出於合法的安全原因。）

KaTeX has no `\par`, so all macros are long by default and `\long` will be ignored.
KaTeX 沒有 `\par` ，因此所有宏預設都是長宏，並且 `\long` 將被忽略。

Available functions include:
可用的功能包括：

`\char` `\mathchoice` `\TextOrMath` `\@ifstar` `\@ifnextchar` `\@firstoftwo` `\@secondoftwo` `\relax` `\expandafter` `\noexpand`
`\char` `\mathchoice` `\TextOrMath` `\@ifstar` `\@ifnextchar` `\@firstoftwo` `\@secondoftwo` `\relax` `\expandafter` `\noexpand`

@ is a valid character for commands, as if `\makeatletter` were in effect.
@ 是命令的有效字符，就像 `\makeatletter` 有效一樣。

---

# Operators ＊ 運算子

## Big Operators ＊ 大運算子?

|                   |                         |                           |                         |
| ----------------- | ----------------------- | ------------------------- | ----------------------- |
| $\sum$ `\sum`     | $\prod$ `\prod`         | $\bigotimes$ `\bigotimes` | $\bigvee$ `\bigvee`     |
| $\int$ `\int`     | $\coprod$ `\coprod`     | $\bigoplus$ `\bigoplus`   | $\bigwedge$ `\bigwedge` |
| $\iint$ `\iint`   | $\intop$ `\intop`       | $\bigodot$ `\bigodot`     | $\bigcap$ `\bigcap`     |
| $\iiint$ `\iiint` | $\smallint$ `\smallint` | $\biguplus$ `\biguplus`   | $\bigcup$ `\bigcup`     |
| $\oint$ `\oint`   | $​\oiint$ `​\oiint`     | $\oiiint$ `\oiiint`       | $\bigsqcup$ `\bigsqcup` |
Direct Input:
∫∬∭∮∏∐∑⋀⋁⋂⋃⨀⨁⨂⨄⨆∫∬∭∮∏∐∑⋀⋁⋂⋃⨀⨁⨂⨄⨆ ∯ ∰

## Binary Operators ＊ 二元運算子

|                         |                                     |                                     |                                       |
| ----------------------- | ----------------------------------- | ----------------------------------- | ------------------------------------- |
| $+$ `+`                 | $\cdot$ `\cdot`                     | $\gtrdot$ `\gtrdot`                 | $x \pmod a$ `x \pmod a`               |
| $-$ `-`                 | $\cdotp$ `\cdotp`                   | $\intercal$ `\intercal`             | $x \pod a$ `x \pod a`                 |
| $/$ `/`                 | $\centerdot$ `\centerdot`           | $\land$ `\land`                     | $\rhd$ `\rhd`                         |
| $*$ `*`                 | $\circ$ `\circ`                     | $\leftthreetimes$ `\leftthreetimes` | $\rightthreetimes$ `\rightthreetimes` |
| $\amalg$ `\amalg`       | $\circledast$ `\circledast`         | $\ldotp$ `\ldotp`                   | $\rtimes$ `\rtimes`                   |
| $\And$ `\And`           | $\circledcirc$ `\circledcirc`       | $\lor$ `\lor`                       | $\setminus$ `\setminus`               |
| $\ast$ `\ast`           | $\circleddash$ `\circleddash`       | $\lessdot$ `\lessdot`               | $\smallsetminus$ `\smallsetminus`     |
| $\barwedge$ `\barwedge` | $\Cup$ `\Cup`                       | $\lhd$ `\lhd`                       | $\sqcap$ `\sqcap`                     |
| $\bigcirc$ `\bigcirc`   | $\cup$ `\cup`                       | $\ltimes$ `\ltimes`                 | $\sqcup$ `\sqcup`                     |
| $\bmod$ `\bmod`         | $\curlyvee$ `\curlyvee`             | $x \mod a$ `x \mod a`               | $\times$ `\times`                     |
| $\boxdot$ `\boxdot`     | $\curlywedge$ `\curlywedge`         | $\mp$ `\mp`                         | $\unlhd$ `\unlhd`                     |
| $\boxminus$ `\boxminus` | $\div$ `\div`                       | $\odot$ `\odot`                     | $\unrhd$ `\unrhd`                     |
| $\boxplus$ `\boxplus`   | $\divideontimes$ `\divideontimes`   | $\ominus$ `\ominus`                 | $\uplus$ `\uplus`                     |
| $\boxtimes$ `\boxtimes` | $\dotplus$ `\dotplus`               | $\oplus$ `\oplus`                   | $\vee$ `\vee`                         |
| $\bullet$ `\bullet`     | $\doublebarwedge$ `\doublebarwedge` | $\otimes$ `\otimes`                 | $\veebar$ `\veebar`                   |
| $\Cap$ `\Cap`           | $\doublecap$ `\doublecap`           | $\oslash$ `\oslash`                 | $\wedge$ `\wedge`                     |
| $\cap$ `\cap`           | $\doublecup$ `\doublecup`           | $\pm$ `\pm`<br/>$\plusmn$ `\plusmn` | $\wr$ `\wr`                           |
Direct Input:
+−/∗⋅∘∙±×÷∓∔∧∨∩∪≀⊎⊓⊔⊕⊖⊗⊘⊙⊚⊛⊝◯∖+−/∗⋅∘∙±×÷∓∔∧∨∩∪≀⊎⊓⊔⊕⊖⊗⊘⊙⊚⊛⊝◯∖

## Fractions and Binomials ＊ 分數和二項式

|                             |                               |                                                             |
| --------------------------- | ----------------------------- | ----------------------------------------------------------- |
| $\frac{a}{b}$ `\frac{a}{b}` | $\tfrac{a}{b}$ `\tfrac{a}{b}` | $\genfrac ( ] {2pt}{1}a{a+1}$ `\genfrac ( ] {2pt}{1}a{a+1}` |
| ${a \over b}$ `{a \over b}` | $\dfrac{a}{b}$ `\dfrac{a}{b}` | ${a \above{2pt} b+1}$ `{a \above{2pt} b+1}`                 |
| $a/b$ `a/b`                 |                               | $\cfrac{a}{1 + \cfrac{1}{b}}$ `\cfrac{a}{1 + \cfrac{1}{b}}` |

|                                 |                                 |                             |
| ------------------------------- | ------------------------------- | --------------------------- |
| $\binom{n}{k}$ `\binom{n}{k}`   | $\dbinom{n}{k}$ `\dbinom{n}{k}` | ${n\brace k}$ `{n\brace k}` |
| ${n \choose k}$ `{n \choose k}` | $\tbinom{n}{k}$ `\tbinom{n}{k}` | ${n\brack k}$ `{n\brack k}` |

## Math Operators ＊ 數學運算子

|                                           |                                                             |                         |                                 |
| ----------------------------------------- | ----------------------------------------------------------- | ----------------------- | ------------------------------- |
| $\arcsin$ `\arcsin`                       | $⁡\cosec$ `⁡\cosec`                                         | $⁡\deg$ `⁡\deg`         | $\sec$ `\sec`                   |
| $⁡\arccos$ `⁡\arccos`                     | $⁡\cosh$ `⁡\cosh`                                           | $⁡\dim$ `⁡\dim`         | $⁡\sin$ `⁡\sin`                 |
| $⁡\arctan$ `⁡\arctan`                     | $⁡\cot$ `⁡\cot`                                             | $⁡\exp$ `⁡\exp`         | $⁡\sinh$ `⁡\sinh`               |
| $⁡\arctg$ `⁡\arctg`                       | $⁡\cotg$ `⁡\cotg`                                           | $⁡\hom$ `⁡\hom`         | $⁡\sh$ `⁡\sh`                   |
| $⁡\arcctg$ `⁡\arcctg`                     | $⁡\coth$ `⁡\coth`                                           | $⁡\ker$ `⁡\ker`         | $⁡\tan$ `⁡\tan`                 |
| $⁡\arg$ `⁡\arg`                           | $⁡\csc$ `⁡\csc`                                             | $⁡\lg$ `⁡\lg`           | $⁡\tanh$ `⁡\tanh`               |
| $⁡\ch⁡$ `⁡\ch⁡`                           | $\ctg$ `\ctg`                                               | $⁡\ln$ `⁡\ln`           | $⁡\tg$ `⁡\tg`                   |
| $⁡\cos$ `⁡\cos`                           | $⁡\cth$ `⁡\cth`                                             | $⁡\log$ `⁡\log`         | $⁡\th$ `⁡\th`                   |
| $⁡\operatorname{f}$ `⁡\operatorname{f}`   |
| $\argmax$ `\argmax`                       | $⁡\injlim$ `⁡\injlim`                                       | $⁡\min$ `⁡\min`         | $⁡​\varinjlim$ `⁡​\varinjlim`   |
| $⁡\argmin$ `⁡\argmin`                     | $⁡\lim$ `⁡\lim`                                             | $⁡\plim$ `⁡\plim`       | $⁡​\varliminf$ `⁡​\varliminf`   |
| $⁡\det$ `⁡\det`                           | $⁡\liminf$ `⁡\liminf`                                       | $⁡\Pr$ `⁡\Pr`           | $⁡\varlimsup$ `⁡\varlimsup`     |
| $⁡\gcd$ `⁡\gcd`                           | $⁡\limsup$ `⁡\limsup`                                       | $⁡\projlim$ `⁡\projlim` | $⁡​\varprojlim$ `⁡​\varprojlim` |
| $⁡\inf$ `⁡\inf`                           | $⁡\max$ `⁡\max`                                             | $⁡\sup$ `⁡\sup`         |                                 |
| $⁡\operatorname*{f}$ `⁡\operatorname*{f}` | $⁡\operatornamewithlimits{f}$ `⁡\operatornamewithlimits{f}` |

Functions in the bottom six rows of this table can take `\limits`.
此表底部六行中的函數可以採用 `\limits` 。

## \sqrt ＊ 開根號

$\sqrt{x}$ `\sqrt{x}`

$\sqrt[3]{x}$ `\sqrt[3]{x}`

---

# Relations ＊ 關係式

=! `\stackrel{!}{=}`

| | | | |
| -------------------------------------- | ----------------------------------- | ---------------- | -------------------------- |
| $=$ `=` | $\doteqdot$ `\doteqdot` | $\lessapprox$ `\lessapprox` | $\smile$ `\smile` |
| $<$ `<` | $\eqcirc$ `\eqcirc` | $\lesseqgtr$ `\lesseqgtr` | $\sqsubset$ `\sqsubset` |
| $>$ `>` | $\eqcolon$ `\eqcolon`<br/>$\minuscolon$ `\minuscolon` | $\lesseqqgtr$ `\lesseqqgtr` | $\sqsubseteq$ `\sqsubseteq` |
| $:$ `:` | $\Eqcolon$ `\Eqcolon`<br/>$\minuscoloncolon$ `\minuscoloncolon` | $\lessgtr$ `\lessgtr` | $\sqsupset$ `\sqsupset` |
| $\approx$ `\approx` | $\eqqcolon$ `\eqqcolon`<br/>$\equalscolon$ `\equalscolon` | $\lesssim$ `\lesssim` | $\sqsupseteq$ `\sqsupseteq` |
| $\approxcolon$ `\approxcolon` | $\Eqqcolon$ `\Eqqcolon`<br/>$\equalscoloncolon$ `\equalscoloncolon` | $\ll$ `\ll` | $\Subset$ `\Subset` |
| $\approxcoloncolon$ `\approxcoloncolon` | $\eqsim$ `\eqsim` | $\lll$ `\lll` | $\subset$ `\subset`<br/>$\sub$ `\sub` |
| $\approxeq$ `\approxeq` | $\eqslantgtr$ `\eqslantgtr` | $\llless$ `\llless` | $\subseteq$ `\subseteq`<br/>$\sube$ `\sube` |
| $\asymp$ `\asymp` | $\eqslantless$ `\eqslantless` | $\lt$ `\lt` | $\subseteqq$ `\subseteqq` |
| $\backepsilon$ `\backepsilon` | $\equiv$ `\equiv` | $\mid$ `\mid` | $\succ$ `\succ` |
| $\backsim$ `\backsim` | $\fallingdotseq$ `\fallingdotseq` | $\models$ `\models` | $\succapprox$ `\succapprox` |
| $\backsimeq$ `\backsimeq` | $\frown$ `\frown` | $\multimap$ `\multimap` | $\succcurlyeq$ `\succcurlyeq` |
| $\between$ `\between` | $\ge$ `\ge` | $\origof$ `\origof` | $\succeq$ `\succeq` |
| $\bowtie$ `\bowtie` | $\geq$ `\geq` | $\owns$ `\owns` | $\succsim$ `\succsim` |
| $\bumpeq$ `\bumpeq` | $\geqq$ `\geqq` | $\parallel$ `\parallel` | $\Supset$ `\Supset` |
| $\Bumpeq$ `\Bumpeq` | $\geqslant$ `\geqslant` | $\perp$ `\perp` | $\supset$ `\supset` |
| $\circeq$ `\circeq` | $\gg$ `\gg` | $\pitchfork$ `\pitchfork` | $\supseteq$ `\supseteq`<br/>$\supe$ `\supe` |
| $\colonapprox$ `\colonapprox` | $\ggg$ `\ggg` | $\prec$ `\prec` | $\supseteqq$ `\supseteqq` |
| $\Colonapprox$ `\Colonapprox`<br/>$\coloncolonapprox$ `\coloncolonapprox` | $\gggtr$ `\gggtr` | $\precapprox$ `\precapprox` | $\thickapprox$ `\thickapprox` |
| $\coloneq$ `\coloneq`<br/>$\colonminus$ `\colonminus` | $\gt$ `\gt` | $\preccurlyeq$ `\preccurlyeq` | $\thicksim$ `\thicksim` |
| $\Coloneq$ `\Coloneq`<br/>$\coloncolonminus$ `\coloncolonminus` | $\gtrapprox$ `\gtrapprox` | $\preceq$ `\preceq` | $\trianglelefteq$ `\trianglelefteq` |
| $\coloneqq$ `\coloneqq`<br/>$\colonequals$ `\colonequals` | $\gtreqless$ `\gtreqless` | $\precsim$ `\precsim` | $\triangleq$ `\triangleq` |
| $\Coloneqq$ `\Coloneqq`<br/>$\coloncolonequals$ `\coloncolonequals` | $\gtreqqless$ `\gtreqqless` | $\propto$ `\propto` | $\trianglerighteq$ `\trianglerighteq` |
| $\colonsim$ `\colonsim` | $\gtrless$ `\gtrless` | $\risingdotseq$ `\risingdotseq` | $\varpropto$ `\varpropto` |
| $\Colonsim$ `\Colonsim`<br/>$\coloncolonsim$ `\coloncolonsim` | $\gtrsim$ `\gtrsim` | $\shortmid$ `\shortmid` | $\vartriangle$ `\vartriangle` |
| $\cong$ `\cong` | $\imageof$ `\imageof` | $\shortparallel$ `\shortparallel` | $\vartriangleleft$ `\vartriangleleft` |
| $\curlyeqprec$ `\curlyeqprec` | $\in$ `\in`<br/>$\isin$ `\isin` | $\sim$ `\sim` | $\vartriangleright$ `\vartriangleright` |
| $\curlyeqsucc$ `\curlyeqsucc` | $\Join$ `\Join` | $\simcolon$ `\simcolon` | $\vcentcolon$ `\vcentcolon`<br/>$\ratio$ `\ratio` |
| $\dashv$ `\dashv` | $\le$ `\le` | $\simcoloncolon$ `\simcoloncolon` | $\vdash$ `\vdash` |
| $\dblcolon$ `\dblcolon`<br/>$\coloncolon$ `\coloncolon` | $\leq$ `\leq` | $\simeq$ `\simeq` | $\vDash$ `\vDash` |
| $\doteq$ `\doteq` | $\leqq$ `\leqq` | $\smallfrown$ `\smallfrown` | $\Vdash$ `\Vdash` |
| $\Doteq$ `\Doteq` | $\leqslant$ `\leqslant` | $\smallsmile$ `\smallsmile` | $\Vvdash$ `\Vvdash` |
Direct Input:
=<>:∈∋∝∼∽≂≃≅≈≊≍≎≏≐≑≒≓≖≗≜≡≤≥≦≧≫≬≳≷≺≻≼≽≾≿⊂⊃⊆⊇⊏⊐⊑⊒⊢⊣⊩⊪⊸⋈⋍⋐⋑⋔⋙⋛⋞⋟⌢⌣⩾⪆⪌⪕⪖⪯⪰⪷⪸⫅⫆≲⩽⪅≶⋚⪋⊥⊨⊶⊷ `≔ ≕ ⩴`

## Negated Relations ＊ 否定關係式

$\not =$ `\not =`

|                           |                                     |                                         |                                   |
| ------------------------- | ----------------------------------- | --------------------------------------- | --------------------------------- |
| $\gnapprox$ `\gnapprox`   | $\ngeqslant$ `\ngeqslant`           | $\nsubseteq$ `\nsubseteq`               | $\precneqq$ `\precneqq`           |
| $\gneq$ `\gneq`           | $\ngtr$ `\ngtr`                     | $\nsubseteqq$ `\nsubseteqq`             | $\precnsim$ `\precnsim`           |
| $\gneqq$ `\gneqq`         | $\nleq$ `\nleq`                     | $\nsucc$ `\nsucc`                       | $\subsetneq$ `\subsetneq`         |
| $\gnsim$ `\gnsim`         | $\nleqq$ `\nleqq`                   | $\nsucceq$ `\nsucceq`                   | $\subsetneqq$ `\subsetneqq`       |
| $\gvertneqq$ `\gvertneqq` | $\nleqslant$ `\nleqslant`           | $\nsupseteq$ `\nsupseteq`               | $\succnapprox$ `\succnapprox`     |
| $\lnapprox$ `\lnapprox`   | $\nless$ `\nless`                   | $\nsupseteqq$ `\nsupseteqq`             | $\succneqq$ `\succneqq`           |
| $\lneq$ `\lneq`           | $\nmid$ `\nmid`                     | $\ntriangleleft$ `\ntriangleleft`       | $\succnsim$ `\succnsim`           |
| $\lneqq$ `\lneqq`         | $\notin$ `\notin`                   | $\ntrianglelefteq$ `\ntrianglelefteq`   | $\supsetneq$ `\supsetneq`         |
| $\lnsim$ `\lnsim`         | $\notni$ `\notni`                   | $\ntriangleright$ `\ntriangleright`     | $\supsetneqq$ `\supsetneqq`       |
| $\lvertneqq$ `\lvertneqq` | $\nparallel$ `\nparallel`           | $\ntrianglerighteq$ `\ntrianglerighteq` | $\varsubsetneq$ `\varsubsetneq`   |
| $\ncong$ `\ncong`         | $\nprec$ `\nprec`                   | $\nvdash$ `\nvdash`                     | $\varsubsetneqq$ `\varsubsetneqq` |
| $\ne$ `\ne`               | $\npreceq$ `\npreceq`               | $\nvDash$ `\nvDash`                     | $\varsupsetneq$ `\varsupsetneq`   |
| $\neq$ `\neq`             | $\nshortmid$ `\nshortmid`           | $\nVDash$ `\nVDash`                     | $\varsupsetneqq$ `\varsupsetneqq` |
| $\ngeq$ `\ngeq`           | $\nshortparallel$ `\nshortparallel` | $\nVdash$ `\nVdash`                     |
| $\ngeqq$ `\ngeqq`         | $\nsim$ `\nsim`                     | $\precnapprox$ `\precnapprox`           |
Direct Input:
/∋∤∦≁≆=≨≩≮≯≰≱⊀⊁⊈⊉⊊⊋⊬⊭⊮⊯⋠⋡⋦⋧⋨⋩⋬⋭⪇⪈⪉⪊⪵⪶⪹⪺⫋⫌

## Arrows ＊ 箭頭

|                                         |                                               |                                           |
| --------------------------------------- | --------------------------------------------- | ----------------------------------------- |
| $\circlearrowleft$ `\circlearrowleft`   | $\leftharpoonup$ `\leftharpoonup`             | $\rArr$ `\rArr`                           |
| $\circlearrowright$ `\circlearrowright` | $\leftleftarrows$ `\leftleftarrows`           | $\rarr$ `\rarr`                           |
| $\curvearrowleft$ `\curvearrowleft`     | $\leftrightarrow$ `\leftrightarrow`           | $\restriction$ `\restriction`             |
| $\curvearrowright$ `\curvearrowright`   | $\Leftrightarrow$ `\Leftrightarrow`           | $\rightarrow$ `\rightarrow`               |
| $\Darr$ `\Darr`                         | $\leftrightarrows$ `\leftrightarrows`         | $\Rightarrow$ `\Rightarrow`               |
| $\dArr$ `\dArr`                         | $\leftrightharpoons$ `\leftrightharpoons`     | $\rightarrowtail$ `\rightarrowtail`       |
| $\darr$ `\darr`                         | $\leftrightsquigarrow$ `\leftrightsquigarrow` | $\rightharpoondown$ `\rightharpoondown`   |
| $\dashleftarrow$ `\dashleftarrow`       | $\Lleftarrow$ `\Lleftarrow`                   | $\rightharpoonup$ `\rightharpoonup`       |
| $\dashrightarrow$ `\dashrightarrow`     | $\longleftarrow$ `\longleftarrow`             | $\rightleftarrows$ `\rightleftarrows`     |
| $\downarrow$ `\downarrow`               | $\Longleftarrow$ `\Longleftarrow`             | $\rightleftharpoons$ `\rightleftharpoons` |
| $\Downarrow$ `\Downarrow`               | $\longleftrightarrow$ `\longleftrightarrow`   | $\rightrightarrows$ `\rightrightarrows`   |
| $\downdownarrows$ `\downdownarrows`     | $\Longleftrightarrow$ `\Longleftrightarrow`   | $\rightsquigarrow$ `\rightsquigarrow`     |
| $\downharpoonleft$ `\downharpoonleft`   | $\longmapsto$ `\longmapsto`                   | $\Rrightarrow$ `\Rrightarrow`             |
| $\downharpoonright$ `\downharpoonright` | $\longrightarrow$ `\longrightarrow`           | $\Rsh$ `\Rsh`                             |
| $\gets$ `\gets`                         | $\Longrightarrow$ `\Longrightarrow`           | $\searrow$ `\searrow`                     |
| $\Harr$ `\Harr`                         | $\looparrowleft$ `\looparrowleft`             | $\swarrow$ `\swarrow`                     |
| $\hArr$ `\hArr`                         | $\looparrowright$ `\looparrowright`           | $\to$ `\to`                               |
| $\harr$ `\harr`                         | $\Lrarr$ `\Lrarr`                             | $\twoheadleftarrow$ `\twoheadleftarrow`   |
| $\hookleftarrow$ `\hookleftarrow`       | $\lrArr$ `\lrArr`                             | $\twoheadrightarrow$ `\twoheadrightarrow` |
| $\hookrightarrow$ `\hookrightarrow`     | $\lrarr$ `\lrarr`                             | $\Uarr$ `\Uarr`                           |
| $\iff$ `\iff`                           | $\Lsh$ `\Lsh`                                 | $\uArr$ `\uArr`                           |
| $\impliedby$ `\impliedby`               | $\mapsto$ `\mapsto`                           | $\uarr$ `\uarr`                           |
| $\implies$ `\implies`                   | $\nearrow$ `\nearrow`                         | $\uparrow$ `\uparrow`                     |
| $\Larr$ `\Larr`                         | $\nleftarrow$ `\nleftarrow`                   | $\Uparrow$ `\Uparrow`                     |
| $\lArr$ `\lArr`                         | $\nLeftarrow$ `\nLeftarrow`                   | $\updownarrow$ `\updownarrow`             |
| $\larr$ `\larr`                         | $\nleftrightarrow$ `\nleftrightarrow`         | $\Updownarrow$ `\Updownarrow`             |
| $\leadsto$ `\leadsto`                   | $\nLeftrightarrow$ `\nLeftrightarrow`         | $\upharpoonleft$ `\upharpoonleft`         |
| $\leftarrow$ `\leftarrow`               | $\nrightarrow$ `\nrightarrow`                 | $\upharpoonright$ `\upharpoonright`       |
| $\Leftarrow$ `\Leftarrow`               | $\nRightarrow$ `\nRightarrow`                 | $\upuparrows$ `\upuparrows`               |
| $\leftarrowtail$ `\leftarrowtail`       | $\nwarrow$ `\nwarrow`                         |
| $\leftharpoondown$ `\leftharpoondown`   | $\Rarr$ `\Rarr`                               |
Direct Input:
←↑→↓↔↕↖↗↘↙↚↛↞↠↢↣↦↩↪↫↬↭↮↰↱↶↷↺↻↼↽↾↾↿⇀⇁⇂⇃⇄⇆⇇⇈⇉⇊⇋⇌⇍⇎⇏⇐⇑⇒⇓⇔⇕⇚⇛⇝⇠⇢⟵⟶⟷⟸⟹⟺⟼ ↽

## Extensible Arrows ＊ 可擴展箭頭

|                                                       |                                                         |
| ----------------------------------------------------- | ------------------------------------------------------- |
| $​\xleftarrow{abc}$ `​\xleftarrow{abc}`               | $\xrightarrow[under]{over}$ `\xrightarrow[under]{over}` |
| $\xLeftarrow{abc}$ `\xLeftarrow{abc}`                 | $​\xRightarrow{abc}$ `​\xRightarrow{abc}`               |
| $\xleftrightarrow{abc}$ `\xleftrightarrow{abc}`       | $\xLeftrightarrow{abc}$ `\xLeftrightarrow{abc}`         |
| $\xhookleftarrow{abc}$ `\xhookleftarrow{abc}`         | $​\xhookrightarrow{abc}$ `​\xhookrightarrow{abc}`       |
| $\xtwoheadleftarrow{abc}$ `\xtwoheadleftarrow{abc}`   | $\xtwoheadrightarrow{abc}$ `\xtwoheadrightarrow{abc}`   |
| $\xleftharpoonup{abc}$ `\xleftharpoonup{abc}`         | $\xrightharpoonup{abc}$ `\xrightharpoonup{abc}`         |
| $\xleftharpoondown{abc}$ `\xleftharpoondown{abc}`     | $\xrightharpoondown{abc}$ `\xrightharpoondown{abc}`     |
| $\xleftrightharpoons{abc}$ `\xleftrightharpoons{abc}` | $\xrightleftharpoons{abc}$ `\xrightleftharpoons{abc}`   |
| $\xtofrom{abc}$ `\xtofrom{abc}`                       | $\xmapsto{abc}$ `\xmapsto{abc}`                         |
| $\xlongequal{abc}$ `\xlongequal{abc}`                 |
Extensible arrows all can take an optional argument in the same manner
可擴展箭頭都可以以相同的方式接受可選參數
as `\xrightarrow[under]{over}`.
如 `\xrightarrow[under]{over}` 。

---

# Special Notation ＊ 特殊符號

## Bra-ket Notation ＊ 括號表示法

|                           |                           |                                                                                           |
| ------------------------- | ------------------------- | ----------------------------------------------------------------------------------------- |
| $\bra{\phi}$ `\bra{\phi}` | $\ket{\psi}$ `\ket{\psi}` | $\braket{\phi \mid \psi}$ `\braket{\phi \mid \psi}`                                       |
| $\Bra{\phi}$ `\Bra{\phi}` | $\Ket{\psi}$ `\Ket{\psi}` | $\Braket{ ϕ \mid \frac{∂^2}{∂ t^2} \mid ψ }$ `\Braket{ ϕ \mid \frac{∂^2}{∂ t^2} \mid ψ }` |

## Style, Color, Size, and Font ＊ 樣式、顏色、大小和字體

### Class Assignment ＊

`\mathbin' '\mathclose' '\mathinner' '\mathop`
`\mathopen' '\mathord' '\mathpunct' '\mathrel`

### Color ＊ 顏色

$\color{red} F=ma$ `\color{red} F=ma`

Note that `\color` acts like a switch. Other color functions expect the content to be a function argument:
注意 `\color` 作用類似開關。其他顏色函數需要將內容作為函數參數：

$\textcolor{orange}{F=ma}$ `\textcolor{orange}{F=ma}`

$\textcolor{#228B22}{F=ma}$ `\textcolor{#228B22}{F=ma}`

$$\colorbox{blue}{$F=ma$}$$ `\colorbox{blue}{$F=ma$}`

$$\fcolorbox{red}{blue}{$F=ma$}$$ `\fcolorbox{red}{blue}{$F=ma$}`

Note that, as in LaTeX, `\colorbox` & `\fcolorbox` renders its third argument as text, so you may want to switch back to math mode with `$` as in the examples above.
請注意，與 LaTeX 一樣， `\colorbox` & `\fcolorbox` 將其第三個參數呈現為文本，因此您可能需要使用 `$` 切換回數學模式，如上例所示。

For color definition, KaTeX color functions will accept the standard HTML [predefined color names](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value#Color_keywords). They will also accept an RGB argument in CSS hexa­decimal style. The "#" is optional before a six-digit specification.
對於顏色定義，KaTeX 顏色函數將接受標準 HTML [預先定義顏色名稱](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value#Color_keywords) 。它們也接受 CSS 十六進位格式的 RGB 參數。六位數字前的「#」是可選的。

### Font ＊ 字體

|                                       |                                       |                                   |
| ------------------------------------- | ------------------------------------- | --------------------------------- |
| $\mathrm{Ab0}$ `\mathrm{Ab0}`         | $\mathbf{Ab0}$ `\mathbf{Ab0}`         | $\mathit{Ab0}$ `\mathit{Ab0}`     |
| $\mathnormal{Ab0}$ `\mathnormal{Ab0}` | $\textbf{Ab0}$ `\textbf{Ab0}`         | $\textit{Ab0}$ `\textit{Ab0}`     |
| $\textrm{Ab0}$ `\textrm{Ab0}`         | $\bf Ab0$ `\bf Ab0`                   | $\it Ab0$ `\it Ab0`               |
| $\rm Ab0$ `\rm Ab0`                   | $\bold{Ab0}$ `\bold{Ab0}`             | $\textup{Ab0}$ `\textup{Ab0}`     |
| $\textnormal{Ab0}$ `\textnormal{Ab0}` | $\boldsymbol{Ab0}$ `\boldsymbol{Ab0}` | $\Bbb{AB}$ `\Bbb{AB}`             |
| $\text{Ab0}$ `\text{Ab0}`             | $\bm{Ab0}$ `\bm{Ab0}`                 | $\mathbb{AB}$ `\mathbb{AB}`       |
| $\mathsf{Ab0}$ `\mathsf{Ab0}`         | $\textmd{Ab0}$ `\textmd{Ab0}`         | $\frak{Ab0}$ `\frak{Ab0}`         |
| $\textsf{Ab0}$ `\textsf{Ab0}`         | $\mathtt{Ab0}$ `\mathtt{Ab0}`         | $\mathfrak{Ab0}$ `\mathfrak{Ab0}` |
| $\sf Ab0$ `\sf Ab0`                   | $\texttt{Ab0}$ `\texttt{Ab0}`         | $\mathcal{AB0}$ `\mathcal{AB0}`   |
|                                       | $\tt Ab0$ `\tt Ab0`                   | $\cal AB0$ `\cal AB0`             |
|                                       |                                       | $\mathscr{AB}$ `\mathscr{AB}`     |

One can stack font family, font weight, and font shape by using the `\textXX` versions of the font functions. So `\textsf{\textbf{H}}` will produce HH. The other versions do not stack, e.g., `\mathsf{\mathbf{H}}` will produce HH.
可以使用 `\textXX` 版本的字型函數來堆疊字型系列、字型粗細和字型形狀。因此 `\textsf{\textbf{H}}` 將產生 HH 。其他版本則無法堆疊，例如 `\mathsf{\mathbf{H}}` 將產生 H 。

In cases where KaTeX fonts do not have a bold glyph, `\pmb` can simulate one. For example, `\pmb{\mu}` renders as : μμ
如果 KaTeX 字型沒有粗體字形， `\pmb` 可以模擬一個。例如， `\pmb{\mu}` 渲染為： μ

### Size ＊ 大小

|                         |                                       |
| ----------------------- | ------------------------------------- |
| $\Huge AB$ `\Huge AB`   | $\normalsize AB$ `\normalsize AB`     |
| $\huge AB$ `\huge AB`   | $\small AB$ `\small AB`               |
| $\LARGE AB$ `\LARGE AB` | $\footnotesize AB$ `\footnotesize AB` |
| $\Large AB$ `\Large AB` | $\scriptsize AB$ `\scriptsize AB`     |
| $\large AB$ `\large AB` | $\tiny AB$ `\tiny AB`                 |

### Style ＊ 樣式

|                                                                                         |
| --------------------------------------------------------------------------------------- |
| $\displaystyle\sum_{i=1}^n$ `\displaystyle\sum_{i=1}^n`                                 |
| $\textstyle\sum_{i=1}^n$ `\textstyle\sum_{i=1}^n`                                       |
| $\scriptstyle x$ `\scriptstyle x` (The size of a first sub/superscript)                 |
| $\scriptscriptstyle x$ `\scriptscriptstyle x` (The size of subsequent sub/superscripts) |
| $\lim\limits_x$ `\lim\limits_x`                                                         |
| $\lim\nolimits_x$ `\lim\nolimits_x`                                                     |
| $\verb!x^2!$ `\verb!x^2!`                                                               |
`\text{…}` will accept nested `$…$` fragments and render them in math mode.
`\text{…}` 將接受嵌套的 `$…$` 片段並以數學模式呈現它們。

---

# Symbols and Punctuation ＊ 符號和標點符號

|                                                         |                                               |                                                 |
| ------------------------------------------------------- | --------------------------------------------- | ----------------------------------------------- |
| $% comment$ `% comment`                                 | $\dots$ `\dots`                               | $\KaTeX$ `\KaTeX`                               |
| $\%$ `\%`                                               | $\cdots$ `\cdots`                             | $​\LaTeX$ `​\LaTeX`                             |
| $\#$ `\#`                                               | $\ddots$ `\ddots`                             | $\TeX$ `\TeX`                                   |
| $\&$ `\&`                                               | $\ldots$ `\ldots`                             | $\nabla$ `\nabla`                               |
| $\_$ `\_`                                               | $\vdots$ `\vdots`                             | $\infty$ `\infty`                               |
| $\text{\textunderscore}$ `\text{\textunderscore}`       | $\dotsb$ `\dotsb`                             | $\infin$ `\infin`                               |
| $\text{--}$ `\text{--}`                                 | $\dotsc$ `\dotsc`                             | $\checkmark$ `\checkmark`                       |
| $\text{\textendash}$ `\text{\textendash}`               | $\dotsi$ `\dotsi`                             | $\dag$ `\dag`                                   |
| $\text{---}$ `\text{---}`                               | $\dotsm$ `\dotsm`                             | $\dagger$ `\dagger`                             |
| $\text{\textemdash}$ `\text{\textemdash}`               | $\dotso$ `\dotso`                             | $\text{\textdagger}$ `\text{\textdagger}`       |
| $\text{\textasciitilde}$ `\text{\textasciitilde}`       | $\sdot$ `\sdot`                               | $\ddag$ `\ddag`                                 |
| $\text{\textasciicircum}$ `\text{\textasciicircum}`     | $\mathellipsis$ `\mathellipsis`               | $\ddagger$ `\ddagger`                           |
| $`$ ```                                                 | $\text{\textellipsis}$ `\text{\textellipsis}` | $\text{\textdaggerdbl}$ `\text{\textdaggerdbl}` |
| $\text{\textquoteleft}$ `\text{\textquoteleft}`         | $\Box$ `\Box`                                 | $\Dagger$ `\Dagger`                             |
| $\lq$ `\lq`                                             | $\square$ `\square`                           | $\angle$ `\angle`                               |
| $\text{\textquoteright}$ `\text{\textquoteright}`       | $\blacksquare$ `\blacksquare`                 | $\measuredangle$ `\measuredangle`               |
| $\rq$ `\rq`                                             | $\triangle$ `\triangle`                       | $\sphericalangle$ `\sphericalangle`             |
| $\text{\textquotedblleft}$ `\text{\textquotedblleft}`   | $\triangledown$ `\triangledown`               | $\top$ `\top`                                   |
| $"$ `"`                                                 | $\triangleleft$ `\triangleleft`               | $\bot$ `\bot`                                   |
| $\text{\textquotedblright}$ `\text{\textquotedblright}` | $\triangleright$ `\triangleright`             | $\$$ `\$`                                       |
| $\colon$ `\colon`                                       | $\bigtriangledown$ `\bigtriangledown`         | $\text{\textdollar}$ `\text{\textdollar}`       |
| $\backprime$ `\backprime`                               | $\bigtriangleup$ `\bigtriangleup`             | $\pounds$ `\pounds`                             |
| $\prime$ `\prime`                                       | $\blacktriangle$ `\blacktriangle`             | $\mathsterling$ `\mathsterling`                 |
| $\text{\textless}$ `\text{\textless}`                   | $\blacktriangledown$ `\blacktriangledown`     | $\text{\textsterling}$ `\text{\textsterling}`   |
| $\text{\textgreater}$ `\text{\textgreater}`             | $\blacktriangleleft$ `\blacktriangleleft`     | $\yen$ `\yen`                                   |
| $\text{\textbar}$ `\text{\textbar}`                     | $\blacktriangleright$ `\blacktriangleright`   | $\surd$ `\surd`                                 |
| $\text{\textbardbl}$ `\text{\textbardbl}`               | $\diamond$ `\diamond`                         | $\degree$ `\degree`                             |
| $\text{\textbraceleft}$ `\text{\textbraceleft}`         | $\Diamond$ `\Diamond`                         | $\text{\textdegree}$ `\text{\textdegree}`       |
| $\text{\textbraceright}$ `\text{\textbraceright}`       | $\lozenge$ `\lozenge`                         | $\mho$ `\mho`                                   |
| $\text{\textbackslash}$ `\text{\textbackslash}`         | $\blacklozenge$ `\blacklozenge`               | $\diagdown$ `\diagdown`                         |
| $\text{\P}$ `\text{\P}`<br/>$\P$ `\P`                   | $\star$ `\star`                               | $\diagup$ `\diagup`                             |
| $\text{\S}$ `\text{\S}`<br/>$\S$ `\S`                   | $\bigstar$ `\bigstar`                         | $\flat$ `\flat`                                 |
| $\text{\sect}$ `\text{\sect}`                           | $\clubsuit$ `\clubsuit`                       | $\natural$ `\natural`                           |
| $\copyright$ `\copyright`                               | $\clubs$ `\clubs`                             | $\sharp$ `\sharp`                               |
| $\circledR$ `\circledR`                                 | $\diamondsuit$ `\diamondsuit`                 | $\heartsuit$ `\heartsuit`                       |
| $\text{\textregistered}$ `\text{\textregistered}`       | $\diamonds$ `\diamonds`                       | $\hearts$ `\hearts`                             |
| $\circledS$ `\circledS`                                 | $\spadesuit$ `\spadesuit`                     | $\spades$ `\spades`                             |
| $\text{\textcircled a}$ `\text{\textcircled a}`         | $\maltese$ `\maltese`                         | $\minuso$ `\minuso`                             |
Direct Input:
§ ¶ £¥∇∞⋅∠∡∢♠♡♢♣♭♮♯✓…⋮⋯⋱!£¥∇∞⋅∠∡∢♠♡♢♣♭♮♯✓…⋮⋯⋱! ‼ ⦵

---

# Units ＊ 單位

In KaTeX, units are proportioned as they are in TeX.
在 KaTeX 中，單位與在 TeX 中一樣按比例排列。
KaTeX units are different than CSS units.
KaTeX 單位與 CSS 單位不同。

| KaTeX Unit | Value                           | KaTeX Unit | Value                          |
| ---------- | ------------------------------- | ---------- | ------------------------------ |
| em         | CSS em                          | bp         | $\frac{1}{72}$​ inch × F × G   |
| ex         | CSS ex                          | pc         | 12 KaTeX pt                    |
| mu         | $\frac{1}{18}$​ CSS em          | dd         | $\frac{​1238}{1157​}$ KaTeX pt |
| pt         | $\frac{1}{72.27}$​ inch × F × G | cc         | $\frac{14856}{1157​}$ KaTeX pt |
| mm         | 1 mm × F × G                    | nd         | $\frac{685}{642}$ KaTeX pt     |
| cm         | 1 cm × F × G                    | nc         | $\frac{1370}{107}$ KaTeX pt    |
| in         | 1 inch × F × G                  | sp         | $\frac{1}{65536}$ KaTeX pt     |

where:  在哪裡：

F = (font size of surrounding HTML text)/(10 pt)
F =（周圍 HTML 文字的字體大小）/（10 pt）

G = 1.21 by default, because KaTeX font-size is normally 1.21 × the surrounding font size. This value [can be overridden](https://katex.org/docs/font#font-size-and-lengths) by the CSS of an HTML page.
預設情況下，G = 1.21，因為 KaTeX 字體大小通常是周圍字體大小的 1.21 倍。此值 [可以被 HTML 頁面的 CSS 覆寫](https://katex.org/docs/font#font-size-and-lengths) 。

The effect of style and size:
款式和尺寸的影響：

| Unit     | textstyle | scriptscript | huge |
| -------- | --------- | ------------ | ---- |
| em or ex |           |              |      |
| mu       |           |              |      |
| others   |           |              |      |
