# Sistema de Encuestas de Satisfacción de Clientes

## Modelo Entidad-Relación (ER)

### Entidades y Atributos

#### Cliente
- **ClienteID** (int): Identificador único del cliente.
- **Nombre** (varchar): Nombre del cliente.
- **Email** (varchar): Email del cliente.
- **Telefono** (varchar): Teléfono del cliente.

#### Encuesta
- **EncuestaID** (int): Identificador único de la encuesta.
- **Fecha** (date): Fecha en la que se realizó la encuesta.
- **ClienteID** (int): Referencia al cliente.

#### Pregunta
- **PreguntaID** (int): Identificador único de la pregunta.
- **Descripcion** (varchar): Descripción de la pregunta.

#### Respuesta
- **RespuestaID** (int): Identificador único de la respuesta.
- **EncuestaID** (int): Referencia a la encuesta.
- **PreguntaID** (int): Referencia a la pregunta.
- **Valor** (varchar): Valor de la respuesta.

### Diagrama del Modelo ER

![Diagrama ER](https://github.com/jose-dev-gt/Encuestas/blob/main/diagER.png)

## Modelo Estrella para Análisis de Datos

### Tabla de Hechos: FactEncuestas
- **FactEncuestaID** (int): Clave primaria.
- **ClienteID** (int), **FechaID** (int), **PreguntaID** (int): Claves foráneas.

### Dimensiones

#### DimCliente
- Detalles del cliente.

#### DimFecha
- **FechaID** (int): Clave primaria.
- **Dia**, **Mes**, **Año**: Desglose de la fecha.

#### DimPregunta
- Detalles de la pregunta.

### Diagrama del Modelo Estrella

![Diagrama Modelo Estrella](https://github.com/jose-dev-gt/Encuestas/blob/main/diagstar.png)

## Scripts SQL para el Sistema de Encuestas de Satisfacción de Clientes

### Creación de Tablas
```sql
CREATE TABLE Clientes (
    ClienteID INT PRIMARY KEY,
    Nombre NVARCHAR(100),
    Email NVARCHAR(100),
    Telefono NVARCHAR(15)
);
CREATE TABLE Encuestas (
    EncuestaID INT PRIMARY KEY,
    Fecha DATE,
    ClienteID INT FOREIGN KEY REFERENCES Clientes(ClienteID)
);
CREATE TABLE Preguntas (
    PreguntaID INT PRIMARY KEY,
    Descripcion NVARCHAR(255)
);
CREATE TABLE Respuestas (
    RespuestaID INT PRIMARY KEY,
    EncuestaID INT FOREIGN KEY REFERENCES Encuestas(EncuestaID),
    PreguntaID INT FOREIGN KEY REFERENCES Preguntas(PreguntaID),
    Valor NVARCHAR(255)
);
```

### Procedimientos Almacenados

#### Agregar Cliente
```sql
CREATE PROCEDURE spAgregarCliente
    @Nombre NVARCHAR(100),
    @Email NVARCHAR(100),
    @Telefono NVARCHAR(15)
AS
BEGIN
    INSERT INTO Clientes (Nombre, Email, Telefono)
    VALUES (@Nombre, @Email, @Telefono);
END;
GO
```

#### Registrar Encuesta
```sql
CREATE PROCEDURE spRegistrarEncuesta
    @Fecha DATE,
    @ClienteID INT
AS
BEGIN
    INSERT INTO Encuestas (Fecha, ClienteID)
    VALUES (@Fecha, @ClienteID);
END;
GO
```

#### Añadir Respuesta
```sql
CREATE PROCEDURE spAgregarRespuesta
    @EncuestaID INT,
    @PreguntaID INT,
    @Valor NVARCHAR(255)
AS
BEGIN
    INSERT INTO Respuestas (EncuestaID, PreguntaID, Valor)
    VALUES (@EncuestaID, @PreguntaID, @Valor);
END;
GO
```

### Disparadores

#### Verificar Respuesta Completa
```sql
CREATE TRIGGER trgVerificarRespuestaCompleta
ON Respuestas
BEFORE INSERT
AS
BEGIN
    IF NOT EXISTS (SELECT * FROM Encuestas WHERE EncuestaID = inserted.EncuestaID) OR
       NOT EXISTS (SELECT * FROM Preguntas WHERE PreguntaID = inserted.PreguntaID) OR
       inserted.Valor IS NULL OR inserted.Valor = ''
    BEGIN
        RAISERROR ('La encuesta, pregunta o respuesta no es válida.', 16, 1);
    END
END;
GO
```
