# dnatech-tlpp-layoutdesigner

Ferramentas TLPP para apoiar o desenho, a conferencia e a calibracao de layouts impressos em Protheus usando `FWMSPrinter`.

O projeto e centrado na classe `TLayoutDesigner`, que desenha grades, reguas, caixas, pontos e textos de apoio sobre relatorios/PDFs. Ela tambem converte coordenadas em milimetros para a unidade interna usada pelo printer, aplica origem configuravel e exporta as marcacoes feitas durante o desenho.

## Estrutura

- `src/core`: codigo principal da classe `TLayoutDesigner` e documentacao da proposta da ferramenta.
- `src/test`: exemplos de uso, teste visual de rotulo e rotina empirica de calibracao.

## Resumo de uso

1. Instancie `TLayoutDesigner` passando o objeto `FWMSPrinter`.
2. Configure escala e origem com `SetScaleXY()` ou `SetPageMm()`.
3. Desenhe grade/regua com `GridMm()` e `RulerMm()`.
4. Marque areas do layout com `MarkMm()`, `BoxMm()`, `PointMm()` ou os metodos de calibracao.
5. Use `MmToPrtX()` e `MmToPrtY()` para posicionar chamadas reais como `Say()`, `SayAlign()`, `Code128()`, `EAN13()` e `FWMsBar()`.

Mais detalhes:

- [src/core/README.md](src/core/README.md)
- [src/test/README.md](src/test/README.md)
