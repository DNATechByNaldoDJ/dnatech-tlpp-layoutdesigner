# TODO — Classe genérica de Designer de Layout para Relatórios

## Objetivo

Criar a classe `TLayoutDesigner` como apoio genérico à configuração visual de layouts em relatórios/PDFs gerados pelo Protheus/TOTVS, não limitada a rótulos.

A classe deve permitir desenhar grade, régua, áreas delimitadas, coordenadas, dimensões e conversões entre unidades internas do `FWMsPrinter`/relatório e milímetros, facilitando a parametrização de campos impressos sem ajustes “no chute”.

Nome sugerido:

```advpl
TLayoutDesigner
```

---

## 1. Renomear/generalizar a classe

- [x] Criar nova classe `TLayoutDesigner`.
- [x] Remover dependência conceitual de “rótulo”.
- [x] Manter métodos genéricos: `Grid`, `GridMm`, `Mark`, `MarkMm`, `Point`, `PointMm`, `Text`, `TextMm`.
- [x] Avaliar compatibilidade: `TRotuloDesigner` removido, pois ainda não está em uso.

Exemplo desejado:

```advpl
oDesign := TLayoutDesigner():New(oPrint)
```

---

## 2. Escalas independentes para X e Y

Implementar conversão separada para eixo horizontal e vertical.

### Novos atributos

```advpl
Data nPxPerMMX
Data nPxPerMMY
```

### Métodos necessários

```advpl
Method SetScale(nUnit,nMM)              // mantém compatibilidade; aplica nos dois eixos
Method SetScaleX(nUnit,nMM)
Method SetScaleY(nUnit,nMM)
Method SetScaleXY(nUnitX,nMMX,nUnitY,nMMY)

Method MmToPxX(nMM)
Method MmToPxY(nMM)
Method PxToMmX(nPx)
Method PxToMmY(nPx)

Method CoordMmToPx(nYmm,nXmm)
Method CoordPxToMm(nRow,nCol)
```

### Regras

- `SetScale(nUnit,nMM)` deve configurar `nPxPerMMX` e `nPxPerMMY` com o mesmo fator.
- `SetScaleX()` deve afetar apenas conversões horizontais.
- `SetScaleY()` deve afetar apenas conversões verticais.
- Validar `nMM > 0` para evitar divisão por zero.
- Manter retorno `Self` nos métodos de configuração.

---

## 3. Marcação de áreas com exibição dupla PX/MM

O método `Mark()` deve exibir informações em unidade interna e em milímetros.

Formato sugerido:

```text
CAMPO
PX: X=120 Y=340 W=180 H=20
MM: X=15.00 Y=42.50 W=22.50 H=2.50
```

### Ajustes

- [x] Usar `PxToMmX()` para `X` e `W`.
- [x] Usar `PxToMmY()` para `Y` e `H`.
- [x] Criar parâmetro/opção para exibir somente PX, somente MM ou ambos.

Sugestão de propriedade:

```advpl
Data cInfoMode // "BOTH", "PX", "MM"
```

---

## 4. Métodos em milímetros

Criar métodos que recebam medidas em mm e internamente convertam para unidade do relatório.

```advpl
Method MarkMm(cName,nYmm,nXmm,nWmm,nHmm)
Method PointMm(cName,nYmm,nXmm)
Method TextMm(cText,nYmm,nXmm)
Method BoxMm(cName,nYmm,nXmm,nWmm,nHmm)
```

Exemplo esperado:

```advpl
oDesign:MarkMm("LOTE",80,25,40,5)
```

Equivale a marcar uma área a 25 mm da esquerda e 80 mm do topo.

---

## 5. Grade e régua

### Métodos desejados

```advpl
Method Grid(nStep,nHeight,nWidth)
Method GridMm(nStepMm,nHeight,nWidth)
Method Ruler(nStep,nHeight,nWidth)
Method RulerMm(nStepMm,nHeight,nWidth)
```

### Regras

- `Grid()` trabalha em unidade interna.
- `GridMm()` recebe o passo em mm e converte separadamente X/Y.
- Exibir numeração discreta nas bordas.
- Evitar poluir demais o relatório.
- Permitir configurar espaçamento maior para texto da régua.

---

## 6. Origem e offsets

Adicionar suporte a deslocamento de origem, útil quando o relatório possui margens, área útil, imagem base ou etiqueta dentro de uma página maior.

### Atributos

```advpl
Data nOriginRow
Data nOriginCol
```

### Métodos

```advpl
Method SetOrigin(nRow,nCol)
Method ResetOrigin()
Method ApplyOrigin(nRow,nCol)
```

### Regra

Todo desenho deve considerar a origem configurada.

Exemplo:

```advpl
oDesign:SetOrigin(nTopRotulo,nLeftRotulo)
oDesign:MarkMm("VALIDADE",40,20,30,5)
```

---

## 7. Registro de campos e exportação

A classe deve registrar as marcações feitas para possível exportação posterior.

### Atributo sugerido

```advpl
Data aItems
```

Cada item pode conter:

```advpl
{
    "name"   : cName,
    "type"   : "MARK",
    "row"    : nRow,
    "col"    : nCol,
    "width"  : nWidth,
    "height" : nHeight,
    "xmm"    : nXmm,
    "ymm"    : nYmm,
    "wmm"    : nWmm,
    "hmm"    : nHmm
}
```

### Métodos desejados

```advpl
Method AddItem(cType,cName,nRow,nCol,nWidth,nHeight)
Method GetItems()
Method ClearItems()
Method ExportJson()
Method ExportText()
```

### Exemplo de saída JSON

```json
{
  "LOTE": {"x":25,"y":80,"w":40,"h":5,"unit":"mm"},
  "EAN13": {"x":120,"y":340,"w":180,"h":60,"unit":"px"}
}
```

---

## 8. Configuração visual

Permitir configurar fonte, tamanho e cores usadas pelo designer.

### Atributos

```advpl
Data oFont
Data nColorGrid
Data nColorBox
Data nColorText
Data nColorPoint
Data lShowGridText
Data lShowMarkInfo
```

### Métodos

```advpl
Method SetFont(cFontName,nSize,lBold)
Method SetColors(nGrid,nBox,nText,nPoint)
Method ShowGridText(lShow)
Method ShowMarkInfo(lShow)
Method SetInfoMode(cMode)
```

---

## 9. Compatibilidade com FWMsPrinter/TReport

A classe deve ser compatível com objetos de impressão que possuam métodos como:


```advpl
Say()
Line()
Box() se existir box, box deve ser impresso primeiro caso contrario uma área em branco irá sobrepor os demais conteúdos. Quando não for possível usar box utilizar Line para simular o box. Line é melhor pois permite cor.
```
### referencia [FWMsPrinter](https://tdn.totvs.com/display/public/framework/FWMsPrinter)


### Verificações

- [ ] Testar com `FWMsPrinter`.
- [ ] Testar com relatório/TReport se aplicável.
- [x] Evitar uso de métodos específicos que não existam nos dois contextos, ou encapsular com métodos internos.

### Métodos internos sugeridos

```advpl
Method DrawLine(nRow1,nCol1,nRow2,nCol2,nColor)
Method DrawBox(nRow1,nCol1,nRow2,nCol2,nColor)
Method DrawText(nRow,nCol,cText,oFont,nColor)
```

---

## 10. Integração sugerida no relatório

Exemplo esperado de uso em qualquer relatório:

```advpl
if lDesigner
    oDesign := TLayoutDesigner():New(oPrint)

    oDesign:SetScaleXY(nWidthPx, nWidthMm, nHeightPx, nHeightMm)
    oDesign:SetOrigin(nTopArea,nLeftArea)
    oDesign:SetInfoMode("BOTH")

    oDesign:GridMm(5,nHeightPx,nWidthPx)
    oDesign:Mark("AREA_UTIL",0,0,nWidthPx,nHeightPx)
endif
```

Para campos:

```advpl
oPrint:Say(nLinLote,nColLote,cLote,oFont)

if lDesigner
    oDesign:Mark("LOTE",nLinLote,nColLote,120,20)
endif
```

Ou usando milímetros:

```advpl
aPos := oDesign:CoordMmToPx(80,25)
oPrint:Say(aPos[1],aPos[2],cLote,oFont)
```

---

## 11. Critérios de aceite

- [ ] A classe compila em TLPP/AdvPL.
- [x] Permite uso com escala única via `SetScale()`.
- [x] Permite uso com escala independente via `SetScaleX()`, `SetScaleY()` e `SetScaleXY()`.
- [x] `Mark()` mostra coordenadas PX e MM corretamente.
- [x] `MarkMm()` converte corretamente para unidade interna.
- [x] `GridMm()` desenha grade proporcional nos dois eixos.
- [x] `SetOrigin()` desloca todos os elementos desenhados.
- [x] A classe não depende de nomes ou regras específicas de rótulo.
- [x] O código pode ser reaproveitado em relatórios diversos.
- [x] Existe exemplo de uso no final do arquivo ou em comentário separado.

---

## 12. Observações importantes

- A unidade interna do `FWMsPrinter` não deve ser assumida como milímetro real.
- O usuário/cliente pode trabalhar em mm, mas o relatório continua imprimindo na unidade interna.
- A calibração deve sempre ser feita informando a largura/altura real da área impressa em mm.
- Para maior precisão, preferir `SetScaleXY()` em vez de `SetScale()`.
- A classe deve ser usada como ferramenta de diagnóstico/configuração, não necessariamente em modo produção.

---

## 13. Backlog futuro

- [ ] Gravar layout em tabela de parâmetros.
- [ ] Carregar coordenadas de JSON/SX6/tabela customizada.
- [ ] Permitir edição visual externa via arquivo JSON.
- [ ] Criar modo “preview” sem imprimir os dados reais.
- [ ] Criar método para desenhar zonas proibidas/áreas reservadas.
- [ ] Criar método para comparar layout atual x layout aprovado.
- [ ] Exportar coordenadas para CSV.
- [ ] Criar legenda automática no rodapé com escala, data/hora e usuário.

---

## 14. Referência

- ADVPL\src\tools\TLayoutDesigner.tlpp

## 15. Primeiro uso

- ADVPL\src\Modulos\PCP\Relatorios\HORPCP02.prw (criar uma cópia para testes)
