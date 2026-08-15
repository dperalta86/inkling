<div align="center">

# inkling

**Markdown, sin el ruido.**

[![status](https://img.shields.io/badge/status-early%20development-blue)]()
[![platform](https://img.shields.io/badge/platform-linux-informational)]()
[![license](https://img.shields.io/badge/license-MIT-green)](./LICENSE)

[English](./README.md) | [Español](./README.es.md)

</div>

<!-- TODO: agregar screenshot o gif del editor acá una vez que haya algo estable para mostrar -->

---

inkling es un editor de escritorio liviano para Markdown. No oculta la sintaxis como un WYSIWYG tradicional, ni te obliga a leerla a los gritos como un editor de texto plano: la atenúa. Los símbolos siguen ahí — `**`, `#`, `` ` `` — pero se retiran visualmente hasta que el cursor los necesita.

El archivo en disco es Markdown puro en todo momento. inkling no agrega metadata propia, no inventa un formato: es una forma distinta de mirar el mismo archivo.

## Por qué

Las herramientas de edición de Markdown suelen forzar una elección: o ves los símbolos y perdés fluidez de lectura, o no los ves y perdés la transparencia de la estructura. inkling apuesta a que no hace falta elegir — la sintaxis puede estar presente sin estar en primer plano.

## Cómo se ve esto en la práctica

- **Soft-render con foco dinámico** — los delimitadores se atenúan lejos del cursor y vuelven a su tamaño normal cuando el cursor los toca. Nunca dejan de ser texto editable.
- **Gutter de estructura** — un margen lateral muestra la jerarquía del documento (headings, listas, bloques de código) sin necesitar leer los símbolos.
- **Input rules** — escribís `## ` y se convierte en heading, `- ` y se convierte en lista, sin salir del flujo de escritura.
- **Local-first** — el archivo vive en tu filesystem. Sin cuenta, sin sync obligatorio, sin lock-in.
- **Liviano** — construido sobre Tauri, no Electron. Binario chico, consumo de RAM bajo.

## Estado del proyecto

En desarrollo activo. La arquitectura y el roadmap completo están en [`ROADMAP.md`](./ROADMAP.md). Por ahora, el foco es Linux — Windows y macOS son una posibilidad a futuro, no un compromiso de la v0.1.

## Instalación

*(Pendiente de primer release — el script de instalación se documenta acá apenas exista un binario para distribuir.)*

## Contribuir

Antes de un PR, mirá [`CONTRIBUTING.md`](./CONTRIBUTING.md) — en particular la parte de DCO (firma de commits), que es un requisito, no una sugerencia.

Los contratos técnicos entre módulos (schema, comandos Tauri, plugins) están congelados en [`CONTRACTS.md`](./CONTRACTS.md). Si tu cambio toca uno de esos contratos, el PR lo va a marcar explícitamente.

## Licencia

[MIT](./LICENSE)
