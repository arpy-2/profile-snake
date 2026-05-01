

---

[![Docusaurus](https://github.com/user-attachments/assets/e8cc112c-b37f-4167-acd8-3ebeb14b9947)](https://app.netlify.com/start/deploy?repository=https://github.com/tu-usuario/tu-repo)



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
