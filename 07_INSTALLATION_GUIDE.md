'''
# Guía de Instalación de DATAPOLIS v4.0

## Requisitos Previos

- Docker y Docker Compose
- Git
- Conexión a Internet

## Pasos de Instalación

1. **Clonar el Repositorio:**
   ```bash
   git clone <URL_DEL_REPOSITORIO>
   cd datapolis
   ```

2. **Configurar Variables de Entorno:**
   - Copia el archivo `.env.example` a `.env`.
   - Edita el archivo `.env` con la configuración de tu base de datos y otros servicios.

3. **Levantar los Contenedores:**
   ```bash
   docker-compose up -d --build
   ```

4. **Instalar Dependencias de Laravel:**
   ```bash
   docker-compose exec app composer install
   ```

5. **Ejecutar Migraciones de la Base de Datos:**
   ```bash
   docker-compose exec app php artisan migrate
   ```

6. **Instalar Dependencias de Frontend:**
   ```bash
   docker-compose exec node npm install
   ```

7. **Compilar los Assets de Frontend:**
   ```bash
   docker-compose exec node npm run dev
   ```

8. **Acceder a la Aplicación:**
   - Abre tu navegador y ve a `http://localhost:8080`.
'''
