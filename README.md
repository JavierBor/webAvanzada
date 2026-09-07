# Respuestas del Laboratorio Evaluado: Git, Angular, CI/CD y Terraform
Por: Javier Bórquez 21.802.563-3 y Diego Valenzuela 21.595.757-8.

**1. ¿Por qué no se recomienda desarrollar directamente sobre main en este laboratorio?**
Desarrollar directamente en `main` impide ejecutar validaciones automáticas previas antes de integrar los cambios. Utilizar una rama separada como `devops/ci-cd` permite validar el código mediante un flujo de Integración Continua (CI) a través de un Pull Request, lo que garantiza que la aplicación se construya correctamente y pase las pruebas antes de fusionarse con la rama principal y estable del repositorio.

**2. ¿Qué problema se evita al utilizar `--skip-git` al crear el proyecto Angular?**
Evita la creación de un segundo repositorio `.git` anidado dentro de la carpeta `frontend/`, dado que Git ya fue inicializado en la raíz del repositorio principal.

**3. ¿Qué verifica `npm run build` en esta etapa del laboratorio?**
Verifica que la aplicación de Angular se compile sin errores y que logre generar los artefactos del proyecto correctamente en el entorno local, asegurando que el proceso funcione de manera exitosa antes de automatizarlo en el pipeline de Integración Continua (CI).

**4. ¿Qué utilidad tiene revisar `git status` o `git diff --cached` antes de realizar un commit?**
Permite comprobar exactamente qué información será versionada en el repositorio. `git status` muestra los archivos modificados, preparados y no rastreados, mientras que `git diff --cached` expone los cambios exactos que ya están preparados para ser incluidos en el próximo commit.

**5. ¿Qué evento activa el workflow `ci.yml`?**
Se activa mediante el evento `pull_request`, específicamente cuando el Pull Request tiene como destino la rama `main`.

**6. En `runs-on: ubuntu-latest`, ¿qué representa `ubuntu-latest`?**
Representa el sistema operativo y el entorno del runner (la máquina virtual que provee GitHub Actions) donde se ejecutarán todos los pasos de ese trabajo; en este caso, la versión más reciente disponible de Ubuntu Linux.

**7. Ordene las etapas de validación que ejecuta el job frontend y explique por qué `npm ci` se ejecuta antes que las pruebas.**
El orden de ejecución es el siguiente:
1. Obtener código
2. Configurar Node.js
3. Instalar dependencias
4. Ejecutar pruebas
5. Construir Angular

El comando `npm ci` se ejecuta antes que las pruebas porque es el responsable de descargar e instalar de forma estricta todas las dependencias definidas en el proyecto. Sin estas dependencias instaladas, el entorno de pruebas no tendría las herramientas necesarias para funcionar.

**8. Después del push, indique qué etapa del pipeline falla y qué ocurre con las etapas siguientes.**
Falla la etapa "Ejecutar pruebas" (`npm test`). En GitHub Actions, cuando un paso falla, el proceso finaliza inmediatamente y devuelve un código de salida de error (`exit code 1`). Como consecuencia, las etapas posteriores (como "Construir Angular") no se ejecutan y el pipeline completo es marcado como fallido.

**9. ¿Debería integrarse este Pull Request a main mientras el pipeline está fallando? Justifique.**
No. Un Pull Request con un pipeline fallido indica que el código contiene errores o rompe la funcionalidad existente. Si se integra a la rama `main`, se propagará el error a la versión estable del proyecto, lo cual invalida el propósito de la Integración Continua (CI) como barrera de seguridad.

**10. Clasifique cada elemento como "versionable", "variable/configuración" o "secreto/no versionable".**
* **package.json:** Versionable
* **API_URL pública:** Variable/configuración
* **AWS_REGION:** Variable/configuración
* **DB_PASSWORD:** Secreto/no versionable
* **API_TOKEN:** Secreto/no versionable
* **terraform.tfstate:** Secreto/no versionable

**11. ¿Por qué una contraseña o token no debe escribirse directamente dentro de `ci.yml`, `cd.yml` o un archivo TypeScript del frontend?**
Porque al quedar guardados en el historial de Git, cualquier persona con acceso al repositorio podrá verlos en texto plano, exponiendo el proyecto a graves vulnerabilidades de seguridad.

**12. Si un secreto real fue incluido en un commit y luego se agrega su archivo a `.gitignore`, ¿queda solucionado el problema? Explique qué acción adicional debe realizarse.**
No, no queda solucionado. Aunque el archivo se ignore para futuros cambios, el secreto seguirá siendo visible en el historial de commits anteriores. La acción obligatoria adicional es revocar o invalidar el secreto inmediatamente en el servicio original y limpiar el historial del repositorio de Git.

**13. ¿Qué diferencia existe entre `terraform validate`, `terraform plan` y `terraform apply`?**
* **terraform validate:** Comprueba localmente que la sintaxis y la estructura de los archivos de configuración sean válidas.
* **terraform plan:** Muestra una simulación de los cambios que se realizarán en la infraestructura, sin hacerlos efectivos.
* **terraform apply:** Ejecuta los cambios proyectados para crear, modificar o destruir la infraestructura real.

**14. ¿Por qué `ci.yml` se activa con `pull_request` y `cd.yml` se activa con `push` sobre main?**
`ci.yml` se activa con `pull_request` porque actúa como una barrera de validación (pruebas y construcción) que debe superarse antes de aceptar el código. En cambio, `cd.yml` se activa con `push` sobre `main` porque asume que cualquier código que se ha fusionado en la rama principal ya está validado y está listo para ser desplegado automáticamente al entorno de staging.

**15. ¿Qué función cumple Terraform dentro de este flujo de CD?**
Actúa como la herramienta de Infraestructura como Código (IaC) encargada de automatizar la preparación y aprovisionamiento del entorno de staging simulado dentro del runner de GitHub Actions, organizando los directorios necesarios y copiando los artefactos compilados del frontend.

**16. ¿Por qué el workflow usa `${{ secrets.DEMO_TOKEN }}` en lugar de escribir el valor directamente?**
Utilizar `${{ secrets.DEMO_TOKEN }}` permite inyectar el valor del secreto de manera dinámica y segura durante la ejecución del pipeline. Esto evita escribir la credencial en texto plano dentro del archivo del flujo de trabajo, previniendo que quede expuesta permanentemente en el historial de versiones del repositorio y garantizando la seguridad del proyecto.
