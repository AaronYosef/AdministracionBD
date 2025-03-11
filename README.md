# Creación Dinámica de Bases de Datos y Aplicación Web de Gestión

## Introducción
Para mejorar la seguridad del entorno de trabajo y evitar el uso de la base de datos `master` y el usuario `sa`, se creó una nueva base de datos y un usuario con permisos controlados. Este procedimiento almacenado permite crear bases de datos de manera dinámica, lo que facilita su integración en una aplicación web.

Se utilizaron herramientas como **Deepseek** y **ChatGPT** para la realización del procedimiento almacenado y el desarrollo de la aplicación web en **ASP.NET Core MVC** con Visual Basic.

---

## Paso 1: Creación de la Base de Datos `AdminDB`
Para gestionar las operaciones de administración, se crea una base de datos llamada `AdminDB`.

```sql
CREATE DATABASE AdminDB;
GO
USE AdminDB;
```

---

## Paso 2: Creación del Esquema de Administración
El esquema `Admin` permite agrupar los procedimientos almacenados de administración, mejorando la organización y seguridad del entorno.

```sql
CREATE SCHEMA Admin AUTHORIZATION dbo;
GO
```

---

## Paso 3: Creación de un Usuario con Permisos Limitados
Se crea un nuevo usuario que solo tendrá permisos de ejecución en el esquema `Admin`.

```sql
CREATE LOGIN UsuarioAdminDB WITH PASSWORD = '12_Lola_34';
CREATE USER UsuarioAdminDB FOR LOGIN UsuarioAdminDB;

GRANT EXECUTE ON SCHEMA::Admin TO UsuarioAdminDB;
```

Para verificar los permisos asignados:

```sql
SELECT * FROM sys.database_role_members WHERE member_principal_id = USER_ID('UsuarioAdminDB');
```

---

## Paso 4: Procedimiento Almacenado para Crear Bases de Datos Dinámicas
El siguiente procedimiento permite crear bases de datos especificando:
- Nombre de la base de datos
- Rutas de almacenamiento de archivos (MDF, LDF, NDF)
- Tamaño inicial y crecimiento de los archivos

```sql
USE AdminDB;
CREATE PROCEDURE CrearBaseDeDatosDinamica
    @NombreBD NVARCHAR(100),
    @RutaMDF NVARCHAR(255),
    @TamanoInicialMDF INT,
    @CrecimientoMDF INT,
    @RutaLDF NVARCHAR(255),
    @TamanoInicialLDF INT,
    @CrecimientoLDF INT,
    @RutaNDF NVARCHAR(255) = NULL,
    @TamanoInicialNDF INT = NULL,
    @CrecimientoNDF INT = NULL,
    @MensajeSalida NVARCHAR(255) OUTPUT
AS
BEGIN
    SET NOCOUNT ON;

    -- Validaciones básicas
    IF @NombreBD IS NULL OR LTRIM(RTRIM(@NombreBD)) = ''
    BEGIN
        SET @MensajeSalida = 'Error: El nombre de la base de datos no puede estar vacío.';
        RETURN;
    END

    IF @RutaMDF IS NULL OR LTRIM(RTRIM(@RutaMDF)) = ''
    BEGIN
        SET @MensajeSalida = 'Error: La ruta del archivo MDF no puede estar vacía.';
        RETURN;
    END

    IF @RutaLDF IS NULL OR LTRIM(RTRIM(@RutaLDF)) = ''
    BEGIN
        SET @MensajeSalida = 'Error: La ruta del archivo LDF no puede estar vacía.';
        RETURN;
    END

    IF @TamanoInicialMDF <= 0 OR @CrecimientoMDF <= 0
        OR @TamanoInicialLDF <= 0 OR @CrecimientoLDF <= 0
    BEGIN
        SET @MensajeSalida = 'Error: Los valores de tamaño y crecimiento deben ser mayores a cero.';
        RETURN;
    END

    -- Validación del Filegroup adicional (si se proporciona)
    IF @RutaNDF IS NOT NULL
    BEGIN
        IF @TamanoInicialNDF <= 0 OR @CrecimientoNDF <= 0
        BEGIN
            SET @MensajeSalida = 'Error: Los valores de tamaño y crecimiento del Filegroup secundario deben ser mayores a cero.';
            RETURN;
        END
    END

    -- Validar si la base de datos ya existe
    IF EXISTS (SELECT 1 FROM sys.databases WHERE name = @NombreBD)
    BEGIN
        SET @MensajeSalida = 'Error: La base de datos ya existe.';
        RETURN;
    END

    DECLARE @SQL NVARCHAR(MAX);

    BEGIN TRY
        SET @SQL = '
        CREATE DATABASE ' + QUOTENAME(@NombreBD) + '
        ON 
        (NAME = ' + QUOTENAME(@NombreBD + '_MDF') + ', 
         FILENAME = ''' + @RutaMDF + ''', 
         SIZE = ' + CAST(@TamanoInicialMDF AS NVARCHAR) + 'MB, 
         FILEGROWTH = ' + CAST(@CrecimientoMDF AS NVARCHAR) + 'MB)'

        IF @RutaNDF IS NOT NULL
        BEGIN
            SET @SQL = @SQL + ',
            FILEGROUP [FG_ADICIONAL] 
            (NAME = ' + QUOTENAME(@NombreBD + '_NDF') + ',
             FILENAME = ''' + @RutaNDF + ''',
             SIZE = ' + CAST(@TamanoInicialNDF AS NVARCHAR) + 'MB,
             FILEGROWTH = ' + CAST(@CrecimientoNDF AS NVARCHAR) + 'MB)'
        END

        SET @SQL = @SQL + '
        LOG ON 
        (NAME = ' + QUOTENAME(@NombreBD + '_LDF') + ', 
         FILENAME = ''' + @RutaLDF + ''', 
         SIZE = ' + CAST(@TamanoInicialLDF AS NVARCHAR) + 'MB, 
         FILEGROWTH = ' + CAST(@CrecimientoLDF AS NVARCHAR) + 'MB);'

        EXEC sp_executesql @SQL;

        SET @MensajeSalida = 'Base de datos creada exitosamente.';
    END TRY
    BEGIN CATCH
        SET @MensajeSalida = 'Error: ' + ERROR_MESSAGE();
    END CATCH
END;
```

---

## Paso 6: Creación de una Aplicación ASP.NET Core MVC con Visual Basic

1. Abre Visual Studio y selecciona **"Crear un nuevo proyecto"**.
2. Elige **"Aplicación web ASP.NET Core MVC"**.
3. Selecciona la plantilla y haz clic en **Crear**.
![Logo de Markdown](imagenes/logo.png)
4. Estructura de Carpetas:
![Logo de Markdown](imagenes/logo.png)
5. Colocar tu cadena de conexión correctamente con el Usuario y Base de Datos que fueron creados anteriormente en el archivo de app.settings.json
![Logo de Markdown](imagenes/logo.png)
6. Crear un archivo modelo en la carpeta models, un archivo controlador en controllers y en views en Home/index.cshtml puedes modificarlo para crear la presentación de tu proyecto, en la carpeta Shared/Layouts.cshtml puedes modificarlo para cambiar el flujo entre páginas , también puedes modificar los estilos, por ultimo crear una carpeta(en este caso Bd) y adentro un archivo llamado Crear.cshtml para realizar la vista para poder crear bases de datos dinámicamente, realizar este último paso para crear más distintas vistas para tu gestor sql.     
