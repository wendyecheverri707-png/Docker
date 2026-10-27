# Módulo 12 · Del modelo al contenedor

**Virtualización y Despliegue en la Nube — ERA 3**

En este laboratorio llevas el modelo de sensación térmica entrenado en Colab hasta una imagen Docker publicada, que funciona igual en cualquier máquina.

| Paso | Dónde | Qué se hace |
|---|---|---|
| 1 | Colab | Entrenar y exportar el modelo |
| 2 | Codespaces | [Probar la API sin Docker](#paso-2--probar-la-api-sin-docker) |
| 3 | Codespaces | [Escribir el Dockerfile, construir la imagen y ejecutar el contenedor](#paso-3--del-código-al-contenedor) |
| 4 | GitHub | [Publicar la imagen y probarla en otra máquina](#paso-4--tu-imagen-en-cualquier-máquina) |

## Estructura del repositorio

```
.
├── .devcontainer/
│   └── devcontainer.json   ← describe la máquina del Codespace (Python + Docker)
├── .github/workflows/
│   └── publicar-imagen.yml ← publica tu imagen sola con cada push (Paso 4)
├── app/
│   ├── main.py             ← la API (FastAPI)
│   ├── modelo.joblib       ← el modelo entrenado en Colab
│   ├── metadata.json       ← ficha técnica del modelo
│   ├── requirements.txt    ← versiones exactas de las librerías
│   └── Dockerfile          ← lo crearás tú en el Paso 3
└── README.md
```

> El repositorio ya trae un modelo de respaldo. Si no entrenaste tu propio modelo en el Paso 1, puedes continuar con ese.

---

## Paso 2 — Probar la API sin Docker

La pregunta de este paso es: **¿funciona mi código?** Todavía no usamos Docker. Así, si algo falla más adelante, sabremos que el problema es del contenedor y no de la API.

### 2.1 Crea tu copia del repositorio

1. En la página de este repositorio, pulsa el botón verde **Use this template → Create a new repository**.
2. Créalo en **tu** cuenta y déjalo **Public**. Así el Codespace usará tu cuota gratuita y, en el Paso 4, tu imagen podrá descargarse desde cualquier máquina.

### 2.2 Abre el Codespace

1. En **tu** repositorio, pulsa el **botón verde Code** (a la derecha, sobre la lista de archivos) → pestaña **Codespaces** → **Create codespace on main**.
2. Espera a que termine de prepararse. La primera vez tarda entre 2 y 5 minutos, porque prepara la máquina e instala las librerías de `requirements.txt`.

Al terminar verás VS Code en el navegador, con el explorador de archivos a la izquierda, esta guía abierta y una terminal abajo.

> No confundas el **botón verde Code** (abre Codespaces) con la **pestaña Code** (arriba a la izquierda, muestra los archivos del repositorio).

### 2.3 Sube tus archivos del Paso 1 (opcional)

Si entrenaste tu propio modelo, arrastra tus tres archivos (`modelo.joblib`, `metadata.json` y `requirements.txt`) a la carpeta `app/` y acepta reemplazar los existentes.

Luego reinstala las librerías, porque tus versiones pueden ser distintas a las del respaldo:

```bash
pip install -r app/requirements.txt
```

### 2.4 Arranca la API

```bash
cd app
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

Debes ver en la terminal:

```
✅ Modelo cargado: Polinómica grado 2
INFO:     Uvicorn running on http://0.0.0.0:8000
```

Aparecerá un aviso abajo a la derecha: **Open in Browser**. Ábrelo. Si no lo ves, ve a la pestaña **Puertos** (junto a **Terminal**), pasa el ratón sobre la dirección del puerto `8000` y pulsa el globo 🌐.

### 2.5 Prueba la API

Agrega **solo** `/docs` al final de la dirección que se abrió y presiona Enter. Verás la documentación interactiva que FastAPI genera sola.

> En la dirección del navegador solo se escribe `/docs`. Las pruebas se hacen con los botones de esa página.

**Prueba 1 — información del modelo**

Abre **GET /** → **Try it out** → **Execute**. Revisa qué modelo está cargado y en qué rangos de temperatura y humedad se entrenó.

**Prueba 2 — una predicción**

1. Abre **POST /predecir** → **Try it out**.
2. En el cuadro de texto deja: `{"temperatura": 30, "humedad": 80}`
3. Pulsa **Execute** y baja hasta **Response body**.

Debes obtener una sensación térmica cercana a **37 °C**. Anota el valor de `atendido_por`: es el nombre de la máquina que respondió. Lo volverás a mirar en el Paso 3.

**Prueba 3 — fuera de rango**

En el mismo cuadro, cambia el 30 por 42: `{"temperatura": 42, "humedad": 80}` → **Execute**.

¿Qué responde la API? Compara la temperatura con el rango de entrenamiento de la Prueba 1. ¿Por qué crees que la API se niega a predecir?

**Prueba 4 — un dato que no es número**

En el mismo cuadro escribe, con las comillas: `{"temperatura": "calor", "humedad": 80}` → **Execute**.

¿El mensaje de error lo escribió el profesor en `main.py`, o lo generó FastAPI solo? Compáralo con el de la Prueba 3.

### 2.6 Detén la API

En la terminal donde corre `uvicorn`, pulsa **Ctrl + C**. Debe aparecer `Shutting down`.

### ✅ Lista de verificación

- [ ] La terminal muestra `✅ Modelo cargado`.
- [ ] `/docs` abre en el navegador.
- [ ] `/predecir` responde ≈ 37 °C para 30 °C y 80 %.
- [ ] Una temperatura fuera de rango devuelve un error **422** con un mensaje claro.
- [ ] Anotaste el valor de `atendido_por`.

### Errores frecuentes

| Síntoma | Causa probable | Solución |
|---|---|---|
| `Error loading ASGI app. Could not import module "main"` | Ejecutaste `uvicorn` fuera de la carpeta `app/` | `cd app` y repite |
| `ModuleNotFoundError: No module named 'sklearn'` (u otra librería) | Las librerías no se instalaron | `pip install -r app/requirements.txt` desde la raíz |
| `InconsistentVersionWarning` o error al cargar `modelo.joblib` | Tu `requirements.txt` no coincide con las versiones con que se entrenó el modelo | Sube los tres archivos del **mismo** Colab y reinstala |
| La página del puerto no abre | El servidor no está corriendo o escucha en `127.0.0.1` | Verifica que usaste `--host 0.0.0.0` |
| `{"detail":"Not Found"}` | Escribiste en la dirección una ruta que no existe, por ejemplo `/post/predecir` o `/doc` | En la dirección solo se agrega `/docs` |
| `json_invalid` | Escribiste un texto sin comillas, por ejemplo `calor` | Los textos van entre comillas: `"calor"` |
| Error 404 o 502 con otra dirección | Estás en la pestaña de **otro** Codespace | Revisa que la dirección empiece con el nombre de **tu** Codespace |
| `Address already in use` | Ya hay otro `uvicorn` corriendo | Ciérralo con Ctrl + C en la otra terminal |

### Para pensar antes del Paso 3

1. El Codespace ya tenía Python y las librerías se instalaron en él. Si un compañero tuviera otra versión de Python o de scikit-learn, ¿obtendría el mismo resultado?
2. ¿Qué tendrías que escribirle a alguien para que ejecute tu API en su propio computador? Esa lista de instrucciones es, casi literalmente, lo que será el **Dockerfile**.

---

## Paso 3 — Del código al contenedor

La pregunta de este paso es: **¿funciona mi contenedor?** Es la misma API del Paso 2, pero ahora empaquetada con su propio Python y sus propias librerías, para que corra igual en cualquier máquina.

Para eso crearás un **Dockerfile**: la receta que Docker sigue, línea por línea, para construir una **imagen**. Al ejecutar esa imagen obtienes un **contenedor**.

> **Imagen** = la plantilla congelada (como una clase). **Contenedor** = una copia en ejecución de esa plantilla (como un objeto).

### 3.1 Crea el archivo en el lugar correcto

1. Verifica que detuviste la API del Paso 2 (**Ctrl + C**). Las dos usan el puerto 8000 y no pueden correr al mismo tiempo.
2. En el explorador de la izquierda, haz **clic derecho sobre la carpeta `app` → Nuevo archivo...**
3. Nómbralo exactamente **`Dockerfile`**: con D mayúscula y **sin extensión** (no `Dockerfile.txt`).

Comprueba en la terminal que quedó junto a los demás archivos:

```bash
cd /workspaces/*/app
ls
```

Debes ver `Dockerfile` junto a `main.py`, `metadata.json`, `modelo.joblib` y `requirements.txt`.

> Si el `Dockerfile` aparece fuera de `app/`, arrástralo con el ratón sobre la carpeta `app`.

### 3.2 Escribe la receta

Copia este contenido en tu `Dockerfile`:

```dockerfile
# 1. Imagen base: la misma versión de Python que usó Colab para entrenar
FROM python:3.13-slim

# 2. Carpeta de trabajo dentro del contenedor
WORKDIR /app

# 3. Primero solo los requisitos (así Docker reutiliza esta capa si no cambian)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 4. Después el código, el modelo y su ficha técnica
COPY main.py modelo.joblib metadata.json ./

# 5. Puerto en el que escucha la API
EXPOSE 8000

# 6. Comando que se ejecuta al arrancar el contenedor
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

El archivo se guarda solo. Si en la pestaña del archivo ves un punto ● en lugar de una X, pulsa **Ctrl + S**.

**Qué hace cada instrucción:**

| Instrucción | Qué hace | En tu Dockerfile |
|---|---|---|
| `FROM` | Elige la imagen base: el sistema y el Python de partida | Python 3.13, el mismo de Colab (míralo en `metadata.json`) |
| `WORKDIR` | Crea una carpeta dentro del contenedor y se ubica en ella | `/app` |
| `COPY` | Copia archivos de tu carpeta al contenedor | Primero `requirements.txt`, después el código y el modelo |
| `RUN` | Ejecuta un comando **mientras se construye** la imagen | Instala las versiones exactas de las librerías |
| `EXPOSE` | Documenta en qué puerto escucha la aplicación | 8000, el mismo del Paso 2 |
| `CMD` | El comando que se ejecuta **cuando arranca** el contenedor | El mismo `uvicorn` del Paso 2, sin `--reload` |

### 3.3 Construye la imagen

Desde la carpeta `app`:

```bash
docker build -t sensacion-api .
```

> ⚠️ **No olvides el punto final**, separado por un espacio. Significa "usa esta carpeta". Sin él, Docker responde `requires 1 argument`.

`-t sensacion-api` le pone nombre a la imagen. La primera vez tarda uno o dos minutos. Verás cada instrucción de tu Dockerfile como un paso: `[1/5]`, `[2/5]`… El más lento es `RUN pip install`.

Cuando termine, comprueba que la imagen existe y mira su tamaño:

```bash
docker images
```

### 3.4 Ejecuta el contenedor

```bash
docker run -d --rm --name api -p 8000:8000 sensacion-api
```

| Parte | Significado |
|---|---|
| `-d` | Lo ejecuta en segundo plano: la terminal queda libre para otros comandos |
| `--rm` | Borra el contenedor cuando lo detengas (la imagen se conserva) |
| `--name api` | Le pone el nombre `api`, para referirte a él en los siguientes comandos |
| `-p 8000:8000` | Conecta el puerto 8000 del Codespace con el 8000 del contenedor |
| `sensacion-api` | La imagen de la que se crea el contenedor |

La terminal responde con un código largo: es el identificador del contenedor.

### 3.5 ¿Quién responde ahora?

1. Abre la API: pestaña **Puertos** → globo 🌐 del puerto `8000` → agrega `/docs` a la dirección.
2. **POST /predecir → Try it out → Execute** con `{"temperatura": 30, "humedad": 80}`.
3. Anota `sensacion_termica` y `atendido_por`.
4. En la terminal, mira los contenedores que están corriendo:
   ```bash
   docker ps
   ```
   Aparece una tabla. La primera columna, **CONTAINER ID**, es un código de 12 caracteres que identifica a tu contenedor.

Compara:

- ¿`sensacion_termica` es **la misma** del Paso 2?
- ¿El valor de `atendido_por` es **igual** al código de la columna **CONTAINER ID**?
- En el Paso 2, `atendido_por` era el nombre de tu Codespace. ¿Por qué cambió?

### 3.6 Mira qué ha hecho el contenedor

```bash
docker logs api
```

Muestra lo que el contenedor ha escrito: el `✅ Modelo cargado` y cada petición que recibió. Fíjate en la dirección de origen de las peticiones: ya no es tu IP, como en el Paso 2, sino `172.17.0.1`, la **red interna de Docker**.

### 3.7 Detén el contenedor

```bash
docker stop api
docker ps
```

`docker stop` responde `api`. Después, `docker ps` debe mostrar la tabla **vacía**: el contenedor se detuvo y, por `--rm`, se borró. La imagen sigue guardada (`docker images`).

### 🧪 Desafío: la caché de Docker

1. Construye otra vez **sin cambiar nada**: `docker build -t sensacion-api .` ¿Cuánto tardó? ¿Qué dice junto a cada paso?
2. Abre `main.py`, cambia el texto de `description=` en `FastAPI(...)` y guarda.
3. Construye de nuevo. ¿Qué pasos se repitieron y cuáles dicen `CACHED`?
4. Explica con tus palabras por qué el Dockerfile copia `requirements.txt` antes que `main.py`. ¿Qué pasaría si copiaras todo en una sola línea al principio?

### Guarda tu trabajo en GitHub

Desde la carpeta `app` (donde ya está tu terminal), escribe estos comandos uno por uno:

```bash
git add Dockerfile
git commit -m "Agrega Dockerfile de la API"
git push
```

- `git add` marca el Dockerfile para subirlo.
- `git commit` lo registra en el historial, con un mensaje.
- `git push` lo sube a tu repositorio en GitHub. Debe terminar con `main -> main`.

**Tu repositorio es tu entrega.** En el Paso 4 verás que este push hizo algo más.

### ✅ Lista de verificación

- [ ] El `Dockerfile` está dentro de `app/`, sin extensión.
- [ ] `docker build` termina sin errores y `docker images` muestra `sensacion-api`.
- [ ] `/predecir` responde la misma sensación térmica que en el Paso 2.
- [ ] `atendido_por` es igual al **CONTAINER ID** de `docker ps`.
- [ ] `docker logs api` muestra `✅ Modelo cargado` y tus peticiones.
- [ ] `docker stop api` detuvo el contenedor.
- [ ] Hiciste `git push` de tu `Dockerfile`.

### Errores frecuentes

| Síntoma | Causa probable | Solución |
|---|---|---|
| `requires 1 argument` | Olvidaste el punto final de `docker build` | `docker build -t sensacion-api .` |
| `failed to read dockerfile: open Dockerfile: no such file or directory` | El `Dockerfile` no está en `app/`, tiene otro nombre (`dockerfile`, `Dockerfile.txt`) o la terminal no está en `app/` | Revisa con `ls` y corrige el nombre o la ubicación |
| `"/modelo.joblib": not found` (u otro archivo) | El archivo no existe en `app/` o está mal escrito en el `COPY` | Compara los nombres del `COPY` con los de `ls` |
| `pip install` falla con un error largo (`No matching distribution found`) | La versión de Python en `FROM` no es la del entrenamiento | Usa la versión de `metadata.json` |
| `The container name "/api" is already in use` | Ya hay un contenedor llamado `api` corriendo | `docker stop api` y vuelve a ejecutar |
| `port is already allocated` o `address already in use` | La API del Paso 2 u otro contenedor ocupa el puerto 8000 | Ctrl + C en la terminal de `uvicorn`, o `docker stop api` |
| El contenedor arranca pero la página no abre | Falta `-p 8000:8000` en `docker run` | Detén el contenedor y ejecútalo de nuevo con el comando completo |
| Los cambios en `main.py` no aparecen | El contenedor usa la imagen **construida antes** del cambio | `docker stop api`, `docker build` y `docker run` de nuevo |
| `git push` no hace nada | El comando quedó escrito sin presionar Enter | Presiona Enter |

### Para pensar antes del Paso 4

1. ¿Qué versión de Python tiene el contenedor? ¿Por qué no importa cuál tenga el Codespace?
2. Ahora tu API solo existe mientras tu Codespace está encendido. ¿Qué necesitarías para que un compañero ejecute **tu imagen** en su computador sin tener tu código?

---

## Paso 4 — Tu imagen en cualquier máquina

La pregunta de este paso es: **¿funciona en otra máquina?** Tu imagen ya está publicada: el push del Paso 3 se encargó de eso. Ahora la vas a encontrar y a ejecutar en una máquina que no tiene tu código.

### 4.1 Mira lo que hizo tu push

1. En tu repositorio de GitHub, abre la **pestaña Actions** (arriba, junto a *Code*).
2. Verás una ejecución con el mensaje de tu commit (*"Agrega Dockerfile de la API"*). Haz clic en ella y luego en **publicar** para ver cada paso.
3. Espera el ✅ verde (uno o dos minutos).

GitHub leyó tu `Dockerfile`, construyó la imagen en sus servidores y la publicó. Es lo mismo que hiciste a mano con `docker build`, pero automático. Las instrucciones están en `.github/workflows/publicar-imagen.yml`: ábrelo y lee el nombre de cada paso.

> **Esto es CI/CD** (integración y entrega continuas): cada `git push` construye y publica una versión nueva sin pasos manuales. Es la forma en que los equipos llevan su código a producción.

También verás una ejecución anterior, la del *Initial commit*, con el aviso de que todavía no había Dockerfile. Es normal.

### 4.2 Encuentra tu imagen

1. Ve a la **pestaña Code** de tu repositorio (arriba a la izquierda).
2. A la derecha, baja hasta la sección **Packages** y haz clic en tu paquete.

En la página del paquete verás:

- La etiqueta **Public**. Si dice *Private*: **Package settings → Change visibility → Public**.
- Un comando listo para copiar, con el nombre exacto de tu imagen, por ejemplo:
  ```
  docker pull ghcr.io/usuario/repositorio:2
  ```

**Deja esta pestaña abierta**: la necesitas en el 4.4.

### 4.3 Abre una máquina limpia

1. En **otra pestaña**, abre [github.com/codespaces](https://github.com/codespaces) → a la izquierda, **Templates** → en la tarjeta **Blank**, pulsa **Use this template**.
2. Se abre un Codespace **vacío**: no tiene tu código, ni tu modelo, ni tus librerías. Tampoco tiene esta guía, así que sigue leyéndola en la pestaña de tu repositorio.

### 4.4 Ejecuta tu imagen

1. En la página de tu paquete, copia el comando `docker pull …` con el botón de copiar.
2. Pégalo en la terminal del Codespace vacío **sin presionar Enter**.
3. Cambia solo el inicio: `docker pull` por `docker run --rm -p 8000:8000`. Debe quedar así, con tu usuario y tu repositorio:
   ```bash
   docker run --rm -p 8000:8000 ghcr.io/usuario/repositorio:2
   ```
4. Presiona Enter.

Primero verás *"Unable to find image … locally"*: esta máquina no tiene la imagen, así que Docker la **descarga** de GitHub y la ejecuta.

Abre `/docs` desde la pestaña **Puertos** y prueba **POST /predecir** con `{"temperatura": 30, "humedad": 80}`. Debe dar la misma sensación térmica de siempre, con un `atendido_por` nuevo.

### 4.5 Limpieza

Detén la API con **Ctrl + C** y borra el Codespace en blanco: [github.com/codespaces](https://github.com/codespaces) → **⋯** junto a su nombre → **Delete**.

### 🧪 Desafío: ejecuta la imagen de un compañero

Pídele a un compañero el comando de su paquete y ejecútalo en un Codespace en blanco. ¿Necesitaste su código? ¿Qué Python y qué librerías usó?

### ✅ Lista de verificación

- [ ] La ejecución en **Actions** terminó con ✅ verde.
- [ ] Tu paquete aparece en **Packages** y es **Public**.
- [ ] La imagen corre en un Codespace en blanco con un solo `docker run`.
- [ ] `/predecir` responde la misma sensación térmica que en los pasos anteriores.

### Errores frecuentes

| Síntoma | Causa probable | Solución |
|---|---|---|
| ❌ rojo en la ejecución de tu Dockerfile | El `Dockerfile` tiene un error | Abre la ejecución, mira el paso en rojo, corrige el `Dockerfile` y vuelve a hacer push |
| No aparece ninguna ejecución nueva en Actions | El push no llegó a GitHub, o no cambió nada dentro de `app/` | Verifica que `app/Dockerfile` está en GitHub; también puedes usar **Run workflow** en Actions |
| `not found` o `manifest unknown` | Nombre de la imagen mal escrito o pegado dos veces | Cópialo de nuevo de la página del paquete; revisa que la línea termine en la versión (`:2`) |
| `denied` o `unauthorized` | El paquete es privado | Hazlo público (4.2) |
| Página 404 de GitHub | La dirección tiene un carácter de más, por ejemplo un punto al final | Revisa la dirección |
| Error 502 al abrir la página | La API no está corriendo en ese Codespace | Ejecuta de nuevo el `docker run` y recarga |

### Para cerrar

1. Compara los tres entornos donde corrió tu modelo: Colab, tu Codespace y el Codespace en blanco. ¿En cuáles tuviste que instalar librerías a mano?
2. Si mañana cambias `main.py` y haces push, ¿qué pasos tendrías que repetir para que un compañero use la versión nueva?

---

### Al terminar la sesión

Detén tu Codespace para no gastar cuota: pulsa **F1** (o **Ctrl + Shift + P**), escribe `Stop Current Codespace` y presiona Enter. También puedes detenerlo desde [github.com/codespaces](https://github.com/codespaces), en el menú **⋯** de tu Codespace. Al final del curso puedes borrarlo desde ahí mismo.
