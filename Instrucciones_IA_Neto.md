# Instrucciones para la IA: preparar un CSV para Neto

Copia este texto en tu asistente de IA (ChatGPT, Claude, Gemini…) junto con el archivo que exportaste de tu banco o
bróker. Adjunta también las dos plantillas, `Plantilla_movimientos.csv` y `Plantilla_inversiones.csv`. Pídele que
te devuelva un archivo CSV que puedas importar en Neto (Ajustes → Copia de seguridad → Importar).

> Privacidad: antes de enviar el archivo, si quieres, borra o cambia los datos que no hacen falta (números de
> cuenta o de tarjeta, nombres de personas en los Bizum). Neto no necesita el IBAN ni el saldo de cada fila.

---

## Lo que tienes que hacer (para la IA)

Convierte el archivo adjunto al formato CSV de Neto. Hay dos formatos: uno para **movimientos** (gastos e
ingresos de cuentas corrientes, tarjetas y efectivo) y otro para **inversiones** (compras, ventas, dividendos,
intereses, comisiones e impuestos de brókers y cripto). Si el archivo tiene de los dos, devuelve dos CSV.

Reglas comunes a los dos:

- Codificación **UTF-8**, separador **coma**, una fila por operación y la **primera fila con los nombres de las
  columnas**, exactamente como en la plantilla.
- Fechas en formato **AAAA-MM-DD** (2026-10-03).
- Números con **punto decimal** y sin separador de miles (1234.56, no 1.234,56).
- Si un texto lleva comas, va entre comillas dobles ("Bar Lola, S.L."). Unas comillas dentro de un texto se
  escriben dobles ("Dijo ""hola""").
- No inventes filas ni importes. Si una fila no se entiende, déjala fuera y dime cuál y por qué.

## 1. Movimientos: `fecha,importe,tipo,cuenta,comercio,categoria,nota`

| Columna | Qué va |
|---|---|
| fecha | Día de la operación (no el de valor si son distintos). |
| importe | **Negativo para gastos**, positivo para ingresos. El signo manda. |
| tipo | `gasto` o `ingreso` (debe coincidir con el signo). |
| cuenta | Nombre corto y siempre igual para la misma cuenta: «Santander», «Revolut», «Efectivo». |
| comercio | Nombre limpio del comercio o de quien paga: «Mercadona», no «COMPRA TARJ 4B*1234 MERCADONA MADRID». |
| categoria | Una categoría en español: Supermercado, Restaurantes, Transporte, Vivienda, Suscripciones, Ocio, Ropa, Salud, Viajes, Compras, Otros… Para ingresos: Sueldo, Devolución, Otros. |
| nota | Opcional: lo que ayude a recordar la operación. |

Reglas de movimientos:

- **Traspasos entre cuentas propias no son gastos ni ingresos**: déjalos fuera (o dímelos aparte). Por ejemplo,
  pasar dinero de la cuenta corriente a la de ahorro, recargar Revolut desde el banco o pagar la tarjeta de
  crédito desde la cuenta.
- Las compras de inversión (aportaciones a un bróker o un fondo) tampoco son gastos: van al CSV de inversiones.
- Una **devolución** de una compra es un importe positivo con la categoría de la compra (por ejemplo, 25.00 con
  categoría Compras y nota «Devolución»).
- Limpia el comercio: quita «COMPRA TARJ», «PAGO MOVIL», números de tarjeta, la ciudad y las fechas pegadas al
  nombre. Un Bizum a una persona: comercio «Bizum» y el motivo en la nota.
- Las comisiones del banco son gastos con categoría «Comisiones bancarias».

Ejemplo (ver `Plantilla_movimientos.csv`):

```csv
fecha,importe,tipo,cuenta,comercio,categoria,nota
2026-10-01,-950.00,gasto,Banco principal,Alquiler piso,Vivienda,Domiciliado
2026-10-02,2400.00,ingreso,Banco principal,Empresa S.L. (nómina),Sueldo,Nómina 09/2026
2026-10-03,-64.30,gasto,Banco principal,Mercadona,Supermercado,
2026-10-08,-38.50,gasto,Tarjeta,"Bar Lola, S.L.",Cafés y restaurantes,Cena con amigos
2026-10-15,25.00,ingreso,Banco principal,Devolución Zara,Compras,Es una devolución
```

## 2. Inversiones: `fecha,cuenta,tipo,nombre,simbolo,cantidad,precio_unitario_eur,importe_eur,comision_eur,nota`

| Columna | Qué va |
|---|---|
| fecha | Día de la operación. |
| cuenta | El bróker o la plataforma, siempre con el mismo nombre: «Trade Republic», «MyInvestor», «Revolut Cripto». |
| tipo | `compra`, `venta`, `dividendo`, `interes`, `comision` o `impuesto`. |
| nombre | Nombre del valor: «iShares Core MSCI World», «Apple», «Bitcoin». |
| simbolo | **ISIN** si lo hay (IE00B4L5Y983); si no, el ticker (AAPL, BTC). Vacío en comisiones generales. |
| cantidad | Participaciones o unidades (positiva también en ventas). Vacía en dividendos, intereses, comisiones e impuestos. |
| precio_unitario_eur | Precio de cada unidad **en euros**. Si la operación fue en otra moneda, conviértelo con el tipo de cambio del día. |
| importe_eur | Compras y ventas: cantidad × precio, sin la comisión. Dividendos e intereses: lo cobrado (positivo). Comisiones e impuestos: **negativo**. |
| comision_eur | La comisión de esa compra o venta (0 si no hubo). |
| nota | Opcional. |

Reglas de inversiones:

- Una recompensa gratis (staking, acciones de regalo) es una `compra` con precio 0 e importe 0. El staking de
  cripto se puede agrupar en una fila por mes y moneda.
- Mover cripto dentro de la misma plataforma (de la cartera al staking y vuelta) no es una operación: déjalo fuera.
- Un split o una fusión: una `venta` de lo antiguo y una `compra` de lo nuevo **al mismo coste**, para que no
  aparezca ganancia.
- Retenciones de dividendos e impuestos sobre fondos (como la Vorabpauschale alemana): `impuesto` en negativo.
- Los pagos con la tarjeta del bróker son gastos: van al CSV de **movimientos**, no aquí.

Ejemplo (ver `Plantilla_inversiones.csv`):

```csv
fecha,cuenta,tipo,nombre,simbolo,cantidad,precio_unitario_eur,importe_eur,comision_eur,nota
2026-01-15,Bróker A,compra,iShares Core MSCI World,IE00B4L5Y983,2.5,96.40,241.00,1.00,Plan mensual
2026-04-28,Cripto B,compra,Bitcoin,BTC,0.00002,0,0,0,Recompensa de staking a coste 0
2026-05-10,Bróker A,dividendo,Apple,AAPL,,,0.21,0,Dividendo neto
2026-06-30,Bróker A,comision,Custodia,,,,-1.00,0,Comisión mensual
2026-08-12,Bróker A,venta,Apple,AAPL,1,225.00,225.00,1.00,
```

## Antes de entregarlo

1. Comprueba que la primera fila es exactamente la de la plantilla.
2. Suma los importes de cada cuenta: deberían cuadrar con la diferencia de saldo del extracto (sin los traspasos
   que dejaste fuera).
3. Dime cuántas filas has convertido, cuáles has dejado fuera y por qué.

Neto avisa al importar de las filas con errores, de las que ya existían (no las duplica) y deja deshacer la
importación entera.
