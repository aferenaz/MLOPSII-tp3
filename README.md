## Mini-TP 3 — Servir el modelo por gRPC

Notebook: [`mini-tp3/mini_tp3_colab_Ferenaz.ipynb`](mini-tp3/mini_tp3_colab_Ferenaz.ipynb)

Expuse mi modelo de la Sesión 1 (`gout-demanda-rf` v1.0.0, forecasting de demanda de Gout) como
servicio gRPC: `scoring.proto` con un campo tipado por cada una de las 11 features, un método unary
(`Predict`), uno de server-streaming (`PredictLote`) y un `Health`. El servidor carga el modelo una
sola vez y valida los mismos rangos que el contrato Pydantic de REST (`INVALID_ARGUMENT`, el
equivalente del 422).

| | gRPC | REST |
|---|---|---|
| latencia por llamada (mediana) | 11.7 ms | 14.0–14.7 ms |
| overhead sobre el modelo (≈11 ms) | 0.7 ms | 3.0–3.7 ms |
| tamaño del request | 48 B | 216 B |
| lote de 63 casos | 40 ms (1 stream) | 1249 ms (63 llamadas) |

**Qué noté:** por llamada la diferencia es chica porque la inferencia domina; gRPC gana en
overhead, en tamaño y, sobre todo, en lotes. Lo usaría para tráfico interno (el pipeline que
puntúa las 9 sucursales × 96 SKUs), y REST en el borde. El costo: un contrato que hay que
versionar y compilar de los dos lados, menos tooling para depurar, el navegador no lo habla
directo, y la validación de rangos que Pydantic daba gratis hay que escribirla a mano.
