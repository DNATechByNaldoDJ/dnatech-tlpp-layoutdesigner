# Testes e calibracao

Esta pasta contem rotinas TLPP para validar visualmente o uso da classe `TLayoutDesigner` e calibrar a relacao entre milimetros, unidade interna do `FWMSPrinter` e resultado fisico/visual no PDF.

## TLDTEST01.tlpp

`TLDTEST01` e um exemplo simplificado de rotulo. Ele cria um PDF em paisagem com area de 200 x 130 mm, instancia `TLayoutDesigner`, configura escala e origem, desenha grade/regua e marca campos como codigo, lote, fabricacao, validade, peso, volume, especie e barras.

O caminho da imagem usado no exemplo e propositalmente ficticio. Se o arquivo existir, a rotina usa `SayBitmap()`; se nao existir, imprime apenas textos explicativos no PDF. Isso permite executar o teste mesmo sem o ativo grafico real.

O exemplo tambem mostra o padrao pretendido para o uso da classe:

- configurar a escala com `SetScaleXY()`;
- deslocar a area util com `SetOrigin()`;
- desenhar referencias com `GridMm()`, `RulerMm()` e `MarkMm()`;
- posicionar textos reais com `MmToPrtY()` e `MmToPrtX()`.

## TLDCALIB.tlpp

`TLD_CALIB` e a rotina empirica de calibracao. Ela gera um PDF 200 x 130 mm com paginas dedicadas a:

- `Say()` e `SayAlign()`;
- `Code128()` e `EAN13()`;
- `FWMsBar()` com variacoes de orientacao e `lCmtr2Pix`.

Cada pagina cria a mesma base visual: grade em milimetros, regua, area de pagina e marcas de referencia. Os metodos de calibracao desenham caixas, cruzes de origem e baselines para que o resultado possa ser medido em uma ferramenta externa, como [Inkscape](https://inkscape.org/pt-br/).

A rotina tambem acumula as marcacoes com `ExportText()`, escreve um arquivo `.txt` com as coordenadas e exibe o conteudo via `EEcView()`(por comodidade). Esse arquivo serve como trilha de conferencia entre:

- coordenadas informadas em milimetros;
- coordenadas convertidas para o printer;
- tamanho nominal passado para cada metodo;
- resultado observado no PDF.

## Como calibrar

1. Compile `src/core/tlayoutdesigner.tlpp` junto com as rotinas desta pasta.
2. Execute `TLD_CALIB`.
3. Abra o PDF gerado e meca pagina, grade, textos e barras em uma ferramenta de medicao.
4. Compare as medidas observadas com o `.txt` exportado.
5. Ajuste escala, origem ou parametros de impressao ate que as medidas reais coincidam com as coordenadas esperadas.
6. Use `TLDTEST01` para validar o comportamento em um layout mais proximo do uso final.

A calibracao deve ser repetida quando mudar o driver, a versao/runtime do Protheus, a forma de geracao do PDF ou qualquer parametro que altere a unidade interna do `FWMSPrinter`.
