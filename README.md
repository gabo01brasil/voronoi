# Voronoi WebGL

Efeito Voronoi interativo em tempo real, renderizado 100% na GPU via WebGL2. Um único arquivo HTML, sem dependências externas.

---

## O que é o efeito Voronoi?

O diagrama de Voronoi divide o plano em regiões (células) ao redor de um conjunto de pontos de controle. Cada pixel pertence à célula do ponto mais próximo. O resultado visual são "territórios" coloridos separados por bordas nítidas.

Nesta implementação:

- O **fragment shader** roda o algoritmo inteiro na GPU: para cada pixel, calcula a distância aos N pontos, identifica o mais próximo e aplica cor.
- As **bordas** são detectadas comparando a distância ao 1º e ao 2º ponto mais próximo — onde elas são quase iguais, estamos na fronteira entre duas células.
- Os pontos se **movem** em órbitas elípticas independentes usando seno/cosseno com fases e velocidades derivadas de um hash do índice do ponto, então o padrão nunca se repete.
- A **interação com o mouse** empurra os pontos para longe do cursor dentro de um raio de ≈22% da altura da tela (força dobrada ao clicar).

---

## Como publicar no GitHub Pages

1. **Crie um repositório** no GitHub (pode ser público ou privado com Pages habilitado).

2. **Faça upload do arquivo** `voronoi.html` para a raiz do repositório (ou para a pasta `/docs`).

3. Acesse **Settings → Pages** no repositório.

4. Em *Source*, selecione:
   - Branch: `main` (ou `master`)
   - Pasta: `/ (root)` (ou `/docs` se colocou lá)

5. Clique em **Save**.

6. Aguarde ~1 min. A URL será algo como:
   ```
   https://<seu-usuario>.github.io/<nome-do-repo>/voronoi.html
   ```

> **Dica:** Se quiser que `voronoi.html` responda na raiz (sem o nome do arquivo na URL), renomeie-o para `index.html`.

---

## Como personalizar

### Mudar o número de pontos padrão

No HTML, localize o slider de pontos e altere o atributo `value`:

```html
<input type="range" id="ptsSlider" min="8" max="256" value="64" step="8">
```

Troque `value="64"` por qualquer múltiplo de 8 entre 8 e 256.

> **Atenção:** valores altos (>128) podem reduzir o framerate em GPUs integradas, pois o shader faz N iterações por pixel.

### Mudar o limite máximo de pontos

O loop no fragment shader tem limite fixo de 256 para evitar compilação dinâmica. Para aumentar além de 256, edite **duas** linhas no shader:

```glsl
// linha do loop:
for (int i = 0; i < 256; i++) {
```
e o atributo `max` do slider:
```html
<input type="range" ... max="256" ...>
```

Troque os dois `256` pelo mesmo valor maior (ex.: `512`).

### Mudar as cores

As cores são geradas via HSV com hue derivado do índice do ponto usando a razão áurea (distribuição uniforme):

```glsl
float hue = h(fi * 0.618033988749);
float sat = 0.70 + h(fi * 1.732050808) * 0.30;
vec3  col = hsv2rgb(vec3(hue, sat, 0.92));
```

- **Saturação fixa:** substitua `sat` por um literal, ex.: `0.9` para cores sempre saturadas.
- **Tons frios apenas:** aplique `hue = 0.55 + h(...) * 0.30` para limitar ao azul/ciano.
- **Escala de cinza:** substitua toda a linha `col` por `vec3(h(fi * 1.234))`.
- **Brilho:** o `0.92` no `hsv2rgb` controla o valor V (0 = preto, 1 = máximo brilho).

### Mudar a velocidade padrão

O slider de velocidade vai de 0 a 3.0× (internamente de 0/10 a 30/10). Para mudar o padrão, altere `value="10"` (= 1.0×):

```html
<input type="range" id="spdSlider" min="0" max="30" value="10" step="1">
```

### Mudar o raio de repulsão do mouse

No shader, a linha:
```glsl
float repR = u_res.y * 0.22;
```
define o raio de influência do mouse como 22% da altura da tela. Aumente para `0.40` para uma área maior.
