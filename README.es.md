<p align="center">
  <img src="docs/screenshots/icon.png" alt="Unifyl" width="128" height="128">
</p>

<h1 align="center">Unifyl</h1>

<p align="center">
  <strong>El gestor de archivos de doble panel con IA para macOS.</strong><br>
  La profundidad de Total Commander. El acabado de ForkLift. Inteligencia integrada.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/macOS-14%2B-000?logo=apple&logoColor=white" alt="macOS 14+">
  <img src="https://img.shields.io/badge/Swift-6.0-F05138?logo=swift&logoColor=white" alt="Swift 6">
  <img src="https://img.shields.io/badge/Apple%20Silicon-Optimized-8E8E93" alt="Apple Silicon">
</p>

<p align="center">
  <a href="https://unifyl.app/es/">Sitio web</a> &middot;
  <a href="https://github.com/goodbug89/Unifyl.app/wiki">Documentación</a> &middot;
  <a href="https://github.com/goodbug89/Unifyl.app/releases/latest">Descargar</a>
</p>

<p align="center">
  <strong>Idioma:</strong>
  <a href="README.md">English</a> &middot;
  <a href="README.ja.md">日本語</a> &middot;
  <a href="README.zh-Hans.md">简体中文</a> &middot;
  <a href="README.de.md">Deutsch</a> &middot;
  <strong>Español</strong> &middot;
  <a href="README.fr.md">Français</a>
</p>

---

![Captura de Unifyl](docs/screenshots/01-dualpane.png)

## Vista general

Unifyl es el gestor de archivos de doble panel para macOS que combina herramientas profesionales con inteligencia artificial local. Tus archivos nunca salen de tu Mac.

- **Doble panel** — Diseños de uno, dos, tres y división libre, pestañas, espacios de trabajo, terminal integrado en cada panel
- **Impulsado por IA** — Búsqueda semántica, renombrado por IA, etiquetado inteligente, detección de duplicados — todo local en Apple Neural Engine
- **Remoto y nube** — FTP, SFTP, WebDAV, SMB, S3, Google Drive, Dropbox, OneDrive, Docker, Kubernetes
- **Más de 50 herramientas profesionales** — Renombrado por lotes, comparación de texto/binario/imagen, sincronización de directorios, fusión a 3 vías, edición masiva de EXIF
- **Archivos como carpetas virtuales** — ZIP, 7z, TAR, GZ, BZ2, XZ, ZSTD, RAR (incluida la detección automática de codificación de ZIP heredados CJK)
- **Teclado primero** — Paleta de comandos, navegación al estilo Vim, más de 120 atajos reasignables
- **15 idiomas** — Inglés, coreano, japonés, chino (simplificado y tradicional), alemán, francés, español, portugués, italiano, ruso, turco, árabe, tailandés, vietnamita
- **Refuerzo CJK** (1.3.0): extracción de archivos `.alz` (`unalz` incorporado), vista previa en línea de `.hwpx`, búsqueda multilingüe de nombres de archivo (Pinyin / Romaji / Hangul-Hanja)
- **Refuerzo CJK ampliado** (1.3.1): marcadores automáticos para carpetas de mensajería/nube (KakaoTalk · LINE · WeChat · iCloud Drive — 12 rutas), plegado ancho completo/medio en la búsqueda, conversión de codificación EUC-KR/Shift-JIS/GBK → UTF-8 (con deshacer), detección heurística de formularios del gobierno coreano

---

## Instalación

Obtén la última versión en [unifyl.app/download](https://github.com/goodbug89/Unifyl.app/releases/latest).

También disponible en el [Mac App Store](https://apps.apple.com/app/unifyl/id6773977314?mt=12) (edición sandbox — consulta las diferencias de funciones en las FAQ).

1. Abre el archivo `.dmg`
2. Arrastra Unifyl a tu carpeta Aplicaciones
3. Inicia y concede acceso a los archivos cuando se solicite

---

## Precios

| | **Free** | **Pro — $24.99** |
|---|---|---|
| Diseños de panel | Doble, triple, división libre (todos) | Doble, triple, división libre (todos) |
| Búsqueda | Básica + Spotlight | Regex completo + búsqueda de contenido |
| Herramientas con IA | — | Todas las funciones de IA |
| Conexiones remotas | FTP, SFTP | Todos los protocolos + almacenamiento en la nube |
| Archivos | ZIP | ZIP, 7z, TAR, GZ, BZ2, XZ, ZSTD, RAR, **ALZ** |

> **Nota sobre las ediciones.** La versión de la Mac App Store funciona en un entorno aislado y no incluye el panel de Git, Docker/Kubernetes, los túneles SSH, el terminal integrado, los visores de puertos y procesos, ni la conversión multimedia basada en ffmpeg — macOS no lo permite allí. Todo lo demás es idéntico, y Pro cuesta el mismo pago único en ambos casos. Elige la descarga directa si necesitas alguno de ellos.

Pago único, sin suscripción. Descarga gratuita en la Mac App Store (Pro mediante compra integrada, vinculada a tu ID de Apple con En familia) o descarga directa (Pro mediante LemonSqueezy, en hasta 3 dispositivos). Prueba Pro gratuita de 14 días al primer inicio.

---

## Atajos de teclado clave

| Atajo | Acción |
|---|---|
| `⌘ + ⇧ + P` | Paleta de comandos |
| `Tab` | Cambiar panel activo |
| `Return` | Abrir archivo / entrar en carpeta |
| `⇧ + Return` | Renombrar |
| `Space` | Vista rápida |
| `⌘ + Delete` | Mover a la papelera |
| `⌃ + G` | Cuadrícula de miniaturas |
| `⌘ + T` | Nueva pestaña |
| `⌘ + 2` | Doble panel |
| `⌘ + K` | Terminal |

Los más de 120 atajos son personalizables en «Ajustes > Atajos de teclado».

---

## Compilar desde el código fuente

```bash
git clone https://github.com/goodbug89/Unifyl.app.git
cd unifyl
make doctor    # Verificar la cadena de herramientas
make gen       # Generar proyecto Xcode
make build     # Compilar todos los paquetes en orden
```

Lista completa de funciones, directrices de contribución y desarrollo de plug-ins en el [README en inglés](README.md).

---

## Licencia

Propietaria. Copyright 2024–2026 Unifyl. Todos los derechos reservados.

<p align="center">
  <a href="https://unifyl.app/es/">Sitio web</a> &middot;
  <a href="https://github.com/goodbug89/Unifyl.app/wiki">Documentación</a>
</p>
