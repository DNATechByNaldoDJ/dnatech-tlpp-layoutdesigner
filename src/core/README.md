# TLayoutDesigner

`tlayoutdesigner.tlpp` contem a classe `TLayoutDesigner`, uma ferramenta auxiliar para desenhar e conferir layouts impressos em Protheus. A proposta da classe e funcionar como uma camada de diagnostico visual sobre `FWMSPrinter`, permitindo transformar um PDF ou relatorio em uma superficie medida, com grade, regua, caixas de referencia e marcacoes exportaveis.

Ela nao substitui o relatorio final. O papel dela e acelerar o desenho de rotulos, formularios e areas fixas de impressao: primeiro a classe revela a geometria real do printer, depois o codigo definitivo pode usar as coordenadas medidas em milimetros.

## Finalidade

- Desenhar grade e regua sobre a area impressa.
- Marcar regioes do layout com nome, posicao e tamanho.
- Converter milimetros para coordenadas internas do `FWMSPrinter`.
- Aplicar origem configuravel para alinhar uma area util dentro da pagina.
- Registrar as marcacoes em memoria e exporta-las em texto ou JSON.
- Apoiar calibracao de `Say()`, `SayAlign()`, codigos de barras e imagens.

## Conceito de escala

O `FWMSPrinter` trabalha com unidades internas que nao devem ser tratadas como milimetros reais sem calibracao. Por isso a classe separa:

- medida logica em milimetros, usada por quem desenha o layout;
- unidade interna do printer, usada nas chamadas de impressao;
- origem da area util, aplicada antes de desenhar ou imprimir.

Os metodos `SetScale()`, `SetScaleX()`, `SetScaleY()` e `SetScaleXY()` definem a relacao entre unidade interna e milimetro. O metodo `SetPageMm()` facilita essa configuracao para uma pagina inteira, usando `MmToPdf()` como base e respeitando informacoes disponiveis em `FWMSPrinter` legado quando aplicavel.

## Recursos principais

- `Grid()` e `GridMm()`: desenham linhas guia horizontais e verticais.
- `Ruler()` e `RulerMm()`: desenham regua com marcadores e textos periodicos.
- `Mark()` e `MarkMm()`: desenham e registram areas identificadas.
- `Box()` e `BoxMm()`: desenham caixas de referencia sem textos de marcacao.
- `Point()` e `PointMm()`: marcam pontos com cruz visual.
- `BaselineMm()` e `CrossMm()`: ajudam a conferir origem e baseline de textos.
- `CalibSay()`, `CalibSayAlign()` e `CalibBoxMm()`: imprimem elementos reais acompanhados de referencias para medicao.
- `MmToPrtX()` e `MmToPrtY()`: convertem coordenadas em milimetros para posicionamento no printer.
- `ExportText()` e `ExportJson()`: exportam as marcacoes acumuladas.

## Fluxo sugerido

```tlpp
oDesign:=TLayoutDesigner():New(oPrint)
oDesign:SetScaleXY(nLarImg,nLarMm,nAltImg,nAltMm)
oDesign:SetOrigin(nTopArea,nLeftArea)
oDesign:SetInfoMode("BOTH")

oDesign:GridMm(5,nAltImg,nLarImg)
oDesign:RulerMm(10,nAltImg,nLarImg)
oDesign:MarkMm("LOTE",80,25,40,5)

oPrint:Say(oDesign:MmToPrtY(80),oDesign:MmToPrtX(25),cLote,oFont)
```

Depois de conferir o PDF gerado, ajuste escala, origem e coordenadas ate que a posicao visual e as medidas reais coincidam. Quando o layout estiver estavel, mantenha no relatorio apenas as chamadas necessarias ou deixe a camada de desenho protegida por uma flag de diagnostico.
