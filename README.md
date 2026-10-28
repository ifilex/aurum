# Aurum

### Momentum temporal y exposición ajustada por volatilidad

Aurum es un Expert Advisor desarrollado en **MQL4 para MetaTrader 4**. Este repositorio comparte el código fuente de la **versión 1.00** para su lectura, revisión y experimentación.

La estrategia combina momentum temporal, estimación de volatilidad diaria y rebalanceo periódico de posiciones.

## Contexto del proyecto

Como neuropsicólogo y desarrollador, me interesa estudiar cómo se transforma información en decisiones explícitas.

Aurum explora una pregunta concreta: ¿cómo traducir la dirección reciente de un activo y la variabilidad de sus retornos en una exposición programable?

El código permite examinar esas reglas y sus limitaciones. No incorpora aprendizaje automático ni un modelo neuropsicológico.

## Funcionamiento

### 1. Momentum temporal

Calcula el retorno entre el precio de la vela diaria actual y el cierre de una cantidad configurable de barras diarias atrás.

Con los umbrales incluidos en esta versión:

- Momentum superior al **2%**: dirección compradora.
- Momentum inferior al **−2%**: dirección vendedora.
- Entre ambos umbrales, incluidos sus extremos: exposición deseada nula.

El período predeterminado es de **30 barras diarias**. Los umbrales están definidos directamente en el código.

### 2. Volatilidad realizada

Calcula retornos diarios simples durante una ventana configurable, estima su desviación estándar y la anualiza multiplicando por la raíz cuadrada de 365.

La ventana predeterminada es de **60 barras diarias**.

El cálculo utiliza la vela diaria en formación, por lo que las métricas pueden cambiar durante el día.

### 3. Exposición deseada

La exposición base se calcula como:

    volatilidad objetivo / volatilidad realizada

Luego se limita por activo, se aplican los filtros y se asigna el signo correspondiente a la dirección.

Si la suma de las exposiciones absolutas supera el máximo total configurado, se reducen proporcionalmente antes de enviar las órdenes.

Esta regla no garantiza que la volatilidad efectiva de la cartera coincida con el objetivo. No incorpora una matriz de correlaciones entre activos.

### 4. Filtros

**Volatilidad:** permite excluir activos cuya volatilidad anualizada estimada esté fuera del intervalo configurado.

**ADX:** esta versión excluye valores de ADX diario comprendidos entre `ADX_Minimo` y `ADX_MaximoChop`, inclusive.

Con los valores predeterminados, bloquea el intervalo **20–35**. Los valores inferiores a 20 o superiores a 35 no son bloqueados por ese filtro.

### 5. Rebalanceo

En cada rebalanceo, el sistema:

1. Calcula y limita las exposiciones deseadas.
2. Intenta cerrar todas las posiciones identificadas con su número mágico.
3. Intenta abrir las nuevas posiciones.

El primer rebalanceo puede ejecutarse en el primer tick después de cargar el EA.

Los siguientes se programan según intervalos de semanas transcurridas. El temporizador utiliza `Time[0]`, correspondiente al período del gráfico donde está instalado; no fuerza internamente un gráfico D1 ni un día fijo de la semana.

Las métricas de la estrategia sí utilizan datos D1.

## Instrumentos

La versión compartida contempla:

- `BTCUSD`: activado por defecto.
- `ETHUSD`: activado por defecto.
- `SOLUSD`: desactivado por defecto.

Busca los nombres base y contempla sufijos detectados desde el símbolo del gráfico.

La disponibilidad y las condiciones de contratación dependen del bróker. El programa no incluye conexión directa a exchanges ni gestión de billeteras.

## Parámetros principales

| Parámetro | Valor predeterminado | Función |
|---|---:|---|
| `MagicNumber` | `4400` | Identificador de las operaciones |
| `PeriodoMomentum` | `30` | Barras diarias del cálculo de momentum |
| `PeriodoVolatilidad` | `60` | Barras diarias para estimar volatilidad |
| `ExposicionObjetivo` | `0.30` | Objetivo de volatilidad usado en la fórmula |
| `ExposicionMaxima` | `1.0` | Tope de exposición deseada por activo |
| `RebalanceoSemanal` | `1` | Intervalo previsto entre rebalanceos, en semanas |
| `UsarFiltroADX` | `true` | Activa la exclusión por intervalo de ADX |
| `ADX_Minimo` | `20.0` | Extremo inferior del intervalo excluido |
| `ADX_MaximoChop` | `35.0` | Extremo superior del intervalo excluido |
| `UsarFiltroVolatilidad` | `true` | Activa el filtro de volatilidad |
| `VolatilidadMinima` | `0.15` | Volatilidad anualizada mínima admitida |
| `VolatilidadMaxima` | `1.50` | Volatilidad anualizada máxima admitida |
| `MaxDrawdownPct` | `25.0` | Umbral para bloquear el rebalanceo |
| `MaxExposicionTotalPct` | `80.0` | Tope de exposición deseada agregada |
| `MostrarPanel` | `true` | Muestra el panel informativo |

Estos valores describen el archivo publicado. No son recomendaciones de operación.

## Drawdown y posiciones abiertas

El EA compara el equity de toda la cuenta con el balance registrado al inicializarse.

Si alcanza el umbral de drawdown, activa un bloqueo persistente durante esa ejecución.

**El bloqueo impide ejecutar el rebalanceo completo, incluidos los cierres de posiciones. No liquida las operaciones existentes.**

Las órdenes se abren sin stop loss ni take profit. Por tanto, el umbral de drawdown no constituye un límite efectivo de pérdida.

Al reinicializar el EA, se restablecen tanto el bloqueo como el balance de referencia.

## Instalación para pruebas

1. Abrir MetaTrader 4.
2. Seleccionar **Archivo → Abrir carpeta de datos**.
3. Copiar `AURUM.mq4` dentro de `MQL4/Experts`.
4. Abrir el archivo en MetaEditor y compilar.
5. Revisar los mensajes del compilador.
6. Actualizar la lista de Expert Advisors.
7. Cargar el EA en un gráfico de una cuenta demo.
8. Verificar los símbolos y la disponibilidad de históricos diarios.
9. Revisar los parámetros y los permisos de trading automatizado.

Para alinear el temporizador con barras diarias, utilizar un gráfico D1. Esto no elimina las demás limitaciones documentadas.

El procesamiento depende de los ticks del gráfico donde se carga el EA.

Evitar compartir el mismo número mágico con otras instancias: la rutina de cierre actúa sobre todas las órdenes que coincidan con ese identificador.

## Estado y limitaciones conocidas

Puntos identificados mediante lectura del código:

- **Datos incompletos:** falta una comprobación integral de históricos. Los retornos omitidos por falta de datos siguen dejando el divisor original en el cálculo de volatilidad.
- **Dimensionamiento:** la conversión de exposición a lotes requiere validación contra las unidades del tick, la moneda de la cuenta y las especificaciones de cada contrato.
- **Lote mínimo:** si el lote calculado resulta inferior al mínimo, se eleva al mínimo permitido. Esto puede superar la exposición deseada.
- **Valor del tick:** si no está disponible, se utiliza `1.0` como alternativa, lo que puede distorsionar el tamaño de posición.
- **Cierres fallidos:** el programa registra errores de cierre, pero continúa intentando abrir nuevas posiciones. Pueden coexistir posiciones anteriores y nuevas.
- **Reinicio:** no reconstruye completamente el estado interno desde las posiciones existentes y puede iniciar un nuevo rebalanceo.
- **Exposición mostrada:** el panel utiliza valores internos asignados tras las aperturas; no recalcula la exposición real desde las posiciones y los precios actuales.
- **Exposición residual:** al cerrar posiciones, los valores internos de exposición no se restablecen siempre, por lo que el panel puede mostrar información desactualizada.
- **Costes:** cada rebalanceo intenta cerrar y reabrir posiciones aunque la dirección siga igual, generando costes de ejecución.
- **Parámetros:** falta validación de períodos, límites y paso de lote antes de los cálculos.

El máximo de exposición se aplica a los objetivos calculados. No garantiza un límite continuo sobre las posiciones reales.

## Resultados y validación

El repositorio no incluye un historial auditado ni una validación independiente de rentabilidad.

Esta documentación se basa en la lectura del código. No acredita compilación, ejecución ni resultados de backtesting.

## Autor

Desarrollado por **Emanuel**, neuropsicólogo y desarrollador del proyecto **WineBOX**.

- https://winebox.cloud
- https://wineboxtool.cloud

## Contribuciones

Se reciben reportes de errores y propuestas mediante Issues y Pull Requests.

Incluir versión de MetaTrader 4, instrumento, período del gráfico, parámetros y mensajes relevantes del registro. No compartir credenciales ni datos personales.

## Licencia

Este proyecto se distribuye bajo la licencia **Apache License 2.0**.

Consultar el archivo [LICENSE](LICENSE) para conocer los términos completos.

## Aviso

Material técnico y educativo. No constituye asesoramiento financiero ni una promesa de resultados.
