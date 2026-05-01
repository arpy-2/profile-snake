<div align="center">
  <h1 align="center">
    Docusaurus
    <br />
    <br />
    <a href="https://docusaurus.io">
      <img src="https://docusaurus.io/img/slash-introducing.svg" alt="Docusaurus">
    </a>
  </h1>
</div>

<p align="center">
  <a href="https://x.com/docusaurus"><img src="https://img.shields.io/twitter/follow/docusaurus.svg?style=social" align="right" alt="Twitter Follow" /></a>
  <a href="#backers" alt="sponsors on Open Collective"><img src="https://opencollective.com/Docusaurus/backers/badge.svg" /></a>
  <a href="#sponsors" alt="Sponsors on Open Collective"><img src="https://opencollective.com/Docusaurus/sponsors/badge.svg" /></a>
  <a href="https://www.npmjs.com/package/@docusaurus/core"><img src="https://img.shields.io/npm/v/@docusaurus/core.svg?style=flat" alt="npm version"></a>
  <a href="https://github.com/facebook/docusaurus/actions/workflows/tests.yml"><img src="https://github.com/facebook/docusaurus/actions/workflows/tests.yml/badge.svg" alt="GitHub Actions status"></a>
  <a href="CONTRIBUTING.md#pull-requests"><img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" alt="PRs Welcome"></a>
  <a href="https://discord.gg/docusaurus"><img src="https://img.shields.io/discord/102860784329052160.svg" align="right" alt="Discord Chat" /></a>
  <a href="https://github.com/prettier/prettier"><img alt="code style: prettier" src="https://img.shields.io/badge/code_style-prettier-ff69b4.svg"></a>
  <a href="#license"><img src="https://img.shields.io/github/license/sourcerer-io/hall-of-fame.svg?colorB=ff0000"></a>
  <a href="https://github.com/facebook/jest"><img src="https://img.shields.io/badge/tested_with-jest-99424f.svg" alt="Tested with Jest"></a>
  <a href="https://argos-ci.com" target="_blank" rel="noreferrer noopener" aria-label="Covered by Argos"><img src="https://argos-ci.com/badge.svg" alt="Covered by Argos" width="133" height="20" /></a>
  <a href="https://gitpod.io/#https://github.com/facebook/docusaurus"><img src="https://img.shields.io/badge/Gitpod-Ready--to--Code-blue?logo=gitpod" alt="Gitpod Ready-to-Code"/></a>
  <a href="https://app.netlify.com/sites/docusaurus-2/deploys"><img src="https://api.netlify.com/api/v1/badges/9e1ff559-4405-4ebe-8718-5e21c0774bc8/deploy-status" alt="Netlify Status"></a>
  <a href="https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Ffacebook%2Fdocusaurus%2Ftree%2Fmain%2Fexamples%2Fclassic&project-name=my-docusaurus-site&repo-name=my-docusaurus-site"><img src="https://vercel.com/button" alt="Deploy with Vercel"/></a>
  <a href="https://app.netlify.com/start/deploy?repository=https://github.com/slorber/docusaurus-starter"><img src="https://www.netlify.com/img/deploy/button.svg" alt="Deploy to Netlify"></a>
</p>


# Guía Técnica: Implementación de GitHub Snake en Modo Oscuro
Esta documentación detalla los pasos seguidos para integrar la animación de la serpiente en el perfil de 'GitHub', garantizando que el diseño sea compatible con temas oscuros y forzando la actualización de la caché del servidor.

<br>

## 1. Requisitos de Configuración del Repositorio
Antes de desplegar el código, es imprescindible ajustar los permisos de seguridad de 'GitHub':

Permisos de Escritura: Dirigirse a ´Settings´ > 'Actions' > 'General'. En la sección 'Workflow permissions', activar 'Read and write permissions'. Sin esto, la acción no podrá crear la rama donde se aloja la imagen.

Privacidad de Actividad: Para que la serpiente procese todo el historial de trabajo (incluyendo repositorios privados), activar la opción 'Private contributions' en los ajustes del gráfico de actividad del perfil de usuario.

<br>

## 2. Automatización con GitHub Actions
El proceso se gestiona mediante un archivo de flujo de trabajo en la ruta '.github/workflows/snake.yml'. Se ha optado por una configuración personalizada para evitar el fondo blanco estándar.

Código del Workflow:

YAML
name: generate animation
on:
  schedule:
    - cron: "0 */24 * * *" 
  workflow_dispatch:
  push:
    branches:
      - master
      - main
jobs:
  generate:
    permissions: 
      contents: write
    runs-on: ubuntu-latest
    timeout-minutes: 5
    
    steps:
      - name: generate snake-dark.svg
        uses: Platane/snk/svg-only@v3
        with:
          github_user_name: ${{ github.repository_owner }}
          outputs: |
            dist/snake-dark.svg?palette=github-dark&color_snake=#3c7dd9&color_dots=#161b22,#0e4429,#006d32,#26a641,#39d353          
      - name: push snake-dark.svg to the output branch
        uses: crazy-max/ghaction-github-pages@v3.1.0
        with:
          target_branch: output
          build_dir: dist
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

<br>
          
## 3. Integración en el Perfil (README.md)
Para renderizar la animación, se debe utilizar el enlace directo a la rama de salida. Al haber renombrado el archivo a 'snake-dark.svg', se evita que 'GitHub' sirva una versión antigua almacenada en su caché ('Camo').

Código a insertar:

Markdown
![Snake](https://raw.githubusercontent.com/TU_USUARIO/TU_REPOSITORIO/output/snake-dark.svg)

<br>

## 4. Notas de Mantenimiento y Troubleshooting
Forzado de actualización: Si tras un cambio de colores la imagen no se actualiza, el primer paso es ejecutar manualmente el 'workflow' desde la pestaña 'Actions' > 'Run workflow'.

Caché del Navegador: En caso de no ver cambios inmediatos, refrescar con 'Ctrl + F5' para ignorar la caché local.

Colores Personalizados: La cadena 'color_dots' permite definir la intensidad del verde. El primer valor '#161b22' controla el color de los días sin actividad, esencial para el acabado en modo oscuro.
