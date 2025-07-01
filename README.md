# Base de Datos - Sistema de Gestión de Citas Clínica Dental LARANA

## 1. Análisis de Requerimientos

### Entidades Clave Identificadas:
- **Usuarios** (Pacientes, Odontólogos, Recepcionistas, Administradores, Asistentes)
- **Citas Médicas** (Programación, estados, horarios)
- **Historiales Clínicos** (Diagnósticos, tratamientos)
- **Servicios Odontológicos** (Tipos de tratamientos)
- **Facturación** (Comprobantes electrónicos)
- **Notificaciones** (SMS, Email, Push)
- **Roles y Permisos** (Control de acceso)
- **Incidencias** (Tickets de soporte técnico)

### Reglas de Negocio:
- Un paciente puede tener múltiples citas
- Una cita pertenece a un solo paciente y un solo odontólogo
- Un historial clínico pertenece a un paciente específico
- Los comprobantes deben validarse con SUNAT
- Las notificaciones se envían automáticamente
- Control de roles diferenciado por tipo de usuario

## 2. Modelo Conceptual

### Entidades y Atributos:

**USUARIOS**
- ID, DNI, Nombres, Apellidos, Fecha_Nacimiento, Género, Dirección, Teléfono, Email, Contraseña, Estado, Fecha_Registro

**ROLES**
- ID, Nombre_Rol, Descripción, Estado

**PERMISOS**
- ID, Nombre_Permiso, Descripción, Módulo

**PACIENTES** (Hereda de USUARIOS)
- Usuario_ID, Número_Historia_Clínica, Alergias, Contacto_Emergencia

**ODONTOLOGOS** (Hereda de USUARIOS)
- Usuario_ID, Número_Colegiatura, Especialidad, Horario_Inicio, Horario_Fin

**SERVICIOS**
- ID, Código_Servicio, Nombre, Descripción, Precio, Duración_Minutos, Estado

**CITAS**
- ID, Paciente_ID, Odontólogo_ID, Servicio_ID, Fecha, Hora_Inicio, Hora_Fin, Estado, Motivo, Observaciones

**HISTORIALES_CLINICOS**
- ID, Paciente_ID, Cita_ID, Diagnóstico, Tratamiento, Observaciones, Fecha_Registro

**FACTURAS**
- ID, Cita_ID, Número_Comprobante, Tipo_Comprobante, Subtotal, IGV, Total, Estado_SUNAT, Fecha_Emisión

**NOTIFICACIONES**
- ID, Usuario_ID, Tipo, Canal, Mensaje, Estado, Fecha_Envío

**INCIDENCIAS**
- ID, Usuario_Reporta_ID, Título, Descripción, Prioridad, Estado, Fecha_Creación

### Relaciones:
- USUARIOS (1:N) CITAS
- USUARIOS (1:1) PACIENTES/ODONTOLOGOS
- CITAS (1:1) HISTORIALES_CLINICOS
- CITAS (1:1) FACTURAS
- USUARIOS (N:M) ROLES (tabla intermedia USUARIO_ROLES)
- ROLES (N:M) PERMISOS (tabla intermedia ROL_PERMISOS)

## 3. Modelo Lógico

### Normalización (3FN):

**USUARIOS**
```
PK: usuario_id
UK: dni, email
- usuario_id (INT, AUTO_INCREMENT)
- dni (VARCHAR(8), NOT NULL, UNIQUE)
- nombres (VARCHAR(100), NOT NULL)
- apellidos (VARCHAR(100), NOT NULL)
- fecha_nacimiento (DATE, NOT NULL)
- genero (ENUM('M','F'), NOT NULL)
- direccion (TEXT)
- telefono (VARCHAR(15))
- email (VARCHAR(100), UNIQUE)
- password_hash (VARCHAR(255), NOT NULL)
- estado (ENUM('ACTIVO','INACTIVO'), DEFAULT 'ACTIVO')
- fecha_registro (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
- fecha_actualizacion (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP)
```

**ROLES**
```
PK: rol_id
- rol_id (INT, AUTO_INCREMENT)
- nombre_rol (VARCHAR(50), NOT NULL, UNIQUE)
- descripcion (TEXT)
- estado (ENUM('ACTIVO','INACTIVO'), DEFAULT 'ACTIVO')
```

**PERMISOS**
```
PK: permiso_id
- permiso_id (INT, AUTO_INCREMENT)
- nombre_permiso (VARCHAR(100), NOT NULL, UNIQUE)
- descripcion (TEXT)
- modulo (VARCHAR(50), NOT NULL)
```

**USUARIO_ROLES** (Tabla intermedia)
```
PK: usuario_id + rol_id
FK: usuario_id -> USUARIOS(usuario_id)
FK: rol_id -> ROLES(rol_id)
- usuario_id (INT, NOT NULL)
- rol_id (INT, NOT NULL)
- fecha_asignacion (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
```

**ROL_PERMISOS** (Tabla intermedia)
```
PK: rol_id + permiso_id
FK: rol_id -> ROLES(rol_id)
FK: permiso_id -> PERMISOS(permiso_id)
- rol_id (INT, NOT NULL)
- permiso_id (INT, NOT NULL)
```

**PACIENTES**
```
PK: paciente_id
FK: usuario_id -> USUARIOS(usuario_id)
- paciente_id (INT, AUTO_INCREMENT)
- usuario_id (INT, NOT NULL, UNIQUE)
- numero_historia_clinica (VARCHAR(20), UNIQUE)
- alergias (TEXT)
- contacto_emergencia (VARCHAR(100))
- telefono_emergencia (VARCHAR(15))
```

**ODONTOLOGOS**
```
PK: odontologo_id
FK: usuario_id -> USUARIOS(usuario_id)
- odontologo_id (INT, AUTO_INCREMENT)
- usuario_id (INT, NOT NULL, UNIQUE)
- numero_colegiatura (VARCHAR(20), NOT NULL, UNIQUE)
- especialidad (VARCHAR(100))
- horario_inicio (TIME, DEFAULT '08:00:00')
- horario_fin (TIME, DEFAULT '18:00:00')
- estado_colegiatura (ENUM('ACTIVO','SUSPENDIDO'), DEFAULT 'ACTIVO')
```

**SERVICIOS**
```
PK: servicio_id
- servicio_id (INT, AUTO_INCREMENT)
- codigo_servicio (VARCHAR(20), NOT NULL, UNIQUE)
- nombre (VARCHAR(100), NOT NULL)
- descripcion (TEXT)
- precio (DECIMAL(10,2), NOT NULL)
- duracion_minutos (INT, DEFAULT 30)
- estado (ENUM('ACTIVO','INACTIVO'), DEFAULT 'ACTIVO')
```

**CITAS**
```
PK: cita_id
FK: paciente_id -> PACIENTES(paciente_id)
FK: odontologo_id -> ODONTOLOGOS(odontologo_id)
FK: servicio_id -> SERVICIOS(servicio_id)
- cita_id (INT, AUTO_INCREMENT)
- paciente_id (INT, NOT NULL)
- odontologo_id (INT, NOT NULL)
- servicio_id (INT, NOT NULL)
- fecha_cita (DATE, NOT NULL)
- hora_inicio (TIME, NOT NULL)
- hora_fin (TIME, NOT NULL)
- estado (ENUM('PROGRAMADA','CONFIRMADA','EN_ATENCION','ATENDIDA','CANCELADA','NO_ASISTIO'), DEFAULT 'PROGRAMADA')
- motivo (TEXT)
- observaciones (TEXT)
- fecha_registro (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
- registrado_por (INT) -- FK a USUARIOS
```

**HISTORIALES_CLINICOS**
```
PK: historial_id
FK: paciente_id -> PACIENTES(paciente_id)
FK: cita_id -> CITAS(cita_id)
FK: odontologo_id -> ODONTOLOGOS(odontologo_id)
- historial_id (INT, AUTO_INCREMENT)
- paciente_id (INT, NOT NULL)
- cita_id (INT, NOT NULL)
- odontologo_id (INT, NOT NULL)
- diagnostico (TEXT, NOT NULL)
- tratamiento_realizado (TEXT, NOT NULL)
- observaciones (TEXT)
- proxima_cita_recomendada (DATE)
- fecha_registro (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
```

**FACTURAS**
```
PK: factura_id
FK: cita_id -> CITAS(cita_id)
- factura_id (INT, AUTO_INCREMENT)
- cita_id (INT, NOT NULL)
- numero_comprobante (VARCHAR(20), NOT NULL, UNIQUE)
- tipo_comprobante (ENUM('BOLETA','FACTURA'), NOT NULL)
- subtotal (DECIMAL(10,2), NOT NULL)
- igv (DECIMAL(10,2), NOT NULL)
- total (DECIMAL(10,2), NOT NULL)
- estado_sunat (ENUM('PENDIENTE','VALIDADO','RECHAZADO'), DEFAULT 'PENDIENTE')
- codigo_sunat (VARCHAR(50))
- fecha_emision (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
- fecha_validacion_sunat (TIMESTAMP NULL)
```

**DETALLE_FACTURAS**
```
PK: detalle_id
FK: factura_id -> FACTURAS(factura_id)
FK: servicio_id -> SERVICIOS(servicio_id)
- detalle_id (INT, AUTO_INCREMENT)
- factura_id (INT, NOT NULL)
- servicio_id (INT, NOT NULL)
- cantidad (INT, DEFAULT 1)
- precio_unitario (DECIMAL(10,2), NOT NULL)
- subtotal (DECIMAL(10,2), NOT NULL)
```

**NOTIFICACIONES**
```
PK: notificacion_id
FK: usuario_id -> USUARIOS(usuario_id)
FK: cita_id -> CITAS(cita_id) [OPCIONAL]
- notificacion_id (INT, AUTO_INCREMENT)
- usuario_id (INT, NOT NULL)
- cita_id (INT, NULL)
- tipo (ENUM('RECORDATORIO','CONFIRMACION','CANCELACION','REPROGRAMACION'), NOT NULL)
- canal (ENUM('SMS','EMAIL','PUSH','WHATSAPP'), NOT NULL)
- mensaje (TEXT, NOT NULL)
- estado (ENUM('PENDIENTE','ENVIADO','FALLIDO'), DEFAULT 'PENDIENTE')
- fecha_programada (TIMESTAMP, NOT NULL)
- fecha_envio (TIMESTAMP NULL)
- respuesta_api (TEXT)
```

**INCIDENCIAS**
```
PK: incidencia_id
FK: usuario_reporta_id -> USUARIOS(usuario_id)
FK: usuario_asignado_id -> USUARIOS(usuario_id)
- incidencia_id (INT, AUTO_INCREMENT)
- usuario_reporta_id (INT, NOT NULL)
- usuario_asignado_id (INT, NULL)
- titulo (VARCHAR(200), NOT NULL)
- descripcion (TEXT, NOT NULL)
- prioridad (ENUM('BAJA','MEDIA','ALTA','CRITICA'), DEFAULT 'MEDIA')
- estado (ENUM('ABIERTO','EN_PROCESO','RESUELTO','CERRADO'), DEFAULT 'ABIERTO')
- fecha_creacion (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
- fecha_cierre (TIMESTAMP NULL)
- solucion (TEXT)
```

**ORDENES_MEDICAS**
```
PK: orden_id
FK: cita_id -> CITAS(cita_id)
FK: odontologo_id -> ODONTOLOGOS(odontologo_id)
- orden_id (INT, AUTO_INCREMENT)
- cita_id (INT, NOT NULL)
- odontologo_id (INT, NOT NULL)
- tipo_orden (ENUM('FARMACIA','LABORATORIO'), NOT NULL)
- descripcion (TEXT, NOT NULL)
- indicaciones (TEXT)
- estado (ENUM('PENDIENTE','ENVIADO','COMPLETADO'), DEFAULT 'PENDIENTE')
- fecha_emision (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
```

**SESIONES** (Control de sesiones JWT)
```
PK: sesion_id
FK: usuario_id -> USUARIOS(usuario_id)
- sesion_id (INT, AUTO_INCREMENT)
- usuario_id (INT, NOT NULL)
- token_jwt (TEXT, NOT NULL)
- fecha_inicio (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
- fecha_expiracion (TIMESTAMP, NOT NULL)
- ip_address (VARCHAR(45))
- user_agent (TEXT)
- estado (ENUM('ACTIVA','EXPIRADA','REVOCADA'), DEFAULT 'ACTIVA')
```

## 4. Modelo Físico (MySQL)

### Script de Creación de Base de Datos:

```sql
-- Crear base de datos
CREATE DATABASE IF NOT EXISTS larana_clinic_db 
CHARACTER SET utf8mb4 
COLLATE utf8mb4_unicode_ci;

USE larana_clinic_db;

-- Tabla USUARIOS
CREATE TABLE usuarios (
    usuario_id INT PRIMARY KEY AUTO_INCREMENT,
    dni VARCHAR(8) NOT NULL UNIQUE,
    nombres VARCHAR(100) NOT NULL,
    apellidos VARCHAR(100) NOT NULL,
    fecha_nacimiento DATE NOT NULL,
    genero ENUM('M','F') NOT NULL,
    direccion TEXT,
    telefono VARCHAR(15),
    email VARCHAR(100) UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    estado ENUM('ACTIVO','INACTIVO') DEFAULT 'ACTIVO',
    fecha_registro TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_actualizacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_dni (dni),
    INDEX idx_email (email),
    INDEX idx_estado (estado)
);

-- Tabla ROLES
CREATE TABLE roles (
    rol_id INT PRIMARY KEY AUTO_INCREMENT,
    nombre_rol VARCHAR(50) NOT NULL UNIQUE,
    descripcion TEXT,
    estado ENUM('ACTIVO','INACTIVO') DEFAULT 'ACTIVO'
);

-- Tabla PERMISOS
CREATE TABLE permisos (
    permiso_id INT PRIMARY KEY AUTO_INCREMENT,
    nombre_permiso VARCHAR(100) NOT NULL UNIQUE,
    descripcion TEXT,
    modulo VARCHAR(50) NOT NULL
);

-- Tabla USUARIO_ROLES
CREATE TABLE usuario_roles (
    usuario_id INT NOT NULL,
    rol_id INT NOT NULL,
    fecha_asignacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    PRIMARY KEY (usuario_id, rol_id),
    FOREIGN KEY (usuario_id) REFERENCES usuarios(usuario_id) ON DELETE CASCADE,
    FOREIGN KEY (rol_id) REFERENCES roles(rol_id) ON DELETE CASCADE
);

-- Tabla ROL_PERMISOS
CREATE TABLE rol_permisos (
    rol_id INT NOT NULL,
    permiso_id INT NOT NULL,
    
    PRIMARY KEY (rol_id, permiso_id),
    FOREIGN KEY (rol_id) REFERENCES roles(rol_id) ON DELETE CASCADE,
    FOREIGN KEY (permiso_id) REFERENCES permisos(permiso_id) ON DELETE CASCADE
);

-- Tabla PACIENTES
CREATE TABLE pacientes (
    paciente_id INT PRIMARY KEY AUTO_INCREMENT,
    usuario_id INT NOT NULL UNIQUE,
    numero_historia_clinica VARCHAR(20) UNIQUE,
    alergias TEXT,
    contacto_emergencia VARCHAR(100),
    telefono_emergencia VARCHAR(15),
    
    FOREIGN KEY (usuario_id) REFERENCES usuarios(usuario_id) ON DELETE CASCADE,
    INDEX idx_historia_clinica (numero_historia_clinica)
);

-- Tabla ODONTOLOGOS
CREATE TABLE odontologos (
    odontologo_id INT PRIMARY KEY AUTO_INCREMENT,
    usuario_id INT NOT NULL UNIQUE,
    numero_colegiatura VARCHAR(20) NOT NULL UNIQUE,
    especialidad VARCHAR(100),
    horario_inicio TIME DEFAULT '08:00:00',
    horario_fin TIME DEFAULT '18:00:00',
    estado_colegiatura ENUM('ACTIVO','SUSPENDIDO') DEFAULT 'ACTIVO',
    
    FOREIGN KEY (usuario_id) REFERENCES usuarios(usuario_id) ON DELETE CASCADE,
    INDEX idx_colegiatura (numero_colegiatura)
);

-- Tabla SERVICIOS
CREATE TABLE servicios (
    servicio_id INT PRIMARY KEY AUTO_INCREMENT,
    codigo_servicio VARCHAR(20) NOT NULL UNIQUE,
    nombre VARCHAR(100) NOT NULL,
    descripcion TEXT,
    precio DECIMAL(10,2) NOT NULL,
    duracion_minutos INT DEFAULT 30,
    estado ENUM('ACTIVO','INACTIVO') DEFAULT 'ACTIVO',
    
    INDEX idx_codigo_servicio (codigo_servicio),
    INDEX idx_estado_servicio (estado)
);

-- Tabla CITAS
CREATE TABLE citas (
    cita_id INT PRIMARY KEY AUTO_INCREMENT,
    paciente_id INT NOT NULL,
    odontologo_id INT NOT NULL,
    servicio_id INT NOT NULL,
    fecha_cita DATE NOT NULL,
    hora_inicio TIME NOT NULL,
    hora_fin TIME NOT NULL,
    estado ENUM('PROGRAMADA','CONFIRMADA','EN_ATENCION','ATENDIDA','CANCELADA','NO_ASISTIO') DEFAULT 'PROGRAMADA',
    motivo TEXT,
    observaciones TEXT,
    fecha_registro TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    registrado_por INT,
    
    FOREIGN KEY (paciente_id) REFERENCES pacientes(paciente_id) ON DELETE CASCADE,
    FOREIGN KEY (odontologo_id) REFERENCES odontologos(odontologo_id) ON DELETE CASCADE,
    FOREIGN KEY (servicio_id) REFERENCES servicios(servicio_id) ON DELETE RESTRICT,
    FOREIGN KEY (registrado_por) REFERENCES usuarios(usuario_id) ON DELETE SET NULL,
    
    INDEX idx_fecha_cita (fecha_cita),
    INDEX idx_estado_cita (estado),
    INDEX idx_odontologo_fecha (odontologo_id, fecha_cita),
    
    -- Restricción para evitar solapamientos
    UNIQUE KEY uk_odontologo_horario (odontologo_id, fecha_cita, hora_inicio)
);

-- Tabla HISTORIALES_CLINICOS
CREATE TABLE historiales_clinicos (
    historial_id INT PRIMARY KEY AUTO_INCREMENT,
    paciente_id INT NOT NULL,
    cita_id INT NOT NULL,
    odontologo_id INT NOT NULL,
    diagnostico TEXT NOT NULL,
    tratamiento_realizado TEXT NOT NULL,
    observaciones TEXT,
    proxima_cita_recomendada DATE,
    fecha_registro TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (paciente_id) REFERENCES pacientes(paciente_id) ON DELETE CASCADE,
    FOREIGN KEY (cita_id) REFERENCES citas(cita_id) ON DELETE CASCADE,
    FOREIGN KEY (odontologo_id) REFERENCES odontologos(odontologo_id) ON DELETE CASCADE,
    
    INDEX idx_paciente_fecha (paciente_id, fecha_registro)
);

-- Tabla FACTURAS
CREATE TABLE facturas (
    factura_id INT PRIMARY KEY AUTO_INCREMENT,
    cita_id INT NOT NULL,
    numero_comprobante VARCHAR(20) NOT NULL UNIQUE,
    tipo_comprobante ENUM('BOLETA','FACTURA') NOT NULL,
    subtotal DECIMAL(10,2) NOT NULL,
    igv DECIMAL(10,2) NOT NULL,
    total DECIMAL(10,2) NOT NULL,
    estado_sunat ENUM('PENDIENTE','VALIDADO','RECHAZADO') DEFAULT 'PENDIENTE',
    codigo_sunat VARCHAR(50),
    fecha_emision TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_validacion_sunat TIMESTAMP NULL,
    
    FOREIGN KEY (cita_id) REFERENCES citas(cita_id) ON DELETE CASCADE,
    INDEX idx_numero_comprobante (numero_comprobante),
    INDEX idx_estado_sunat (estado_sunat)
);

-- Tabla DETALLE_FACTURAS
CREATE TABLE detalle_facturas (
    detalle_id INT PRIMARY KEY AUTO_INCREMENT,
    factura_id INT NOT NULL,
    servicio_id INT NOT NULL,
    cantidad INT DEFAULT 1,
    precio_unitario DECIMAL(10,2) NOT NULL,
    subtotal DECIMAL(10,2) NOT NULL,
    
    FOREIGN KEY (factura_id) REFERENCES facturas(factura_id) ON DELETE CASCADE,
    FOREIGN KEY (servicio_id) REFERENCES servicios(servicio_id) ON DELETE RESTRICT
);

-- Tabla NOTIFICACIONES
CREATE TABLE notificaciones (
    notificacion_id INT PRIMARY KEY AUTO_INCREMENT,
    usuario_id INT NOT NULL,
    cita_id INT NULL,
    tipo ENUM('RECORDATORIO','CONFIRMACION','CANCELACION','REPROGRAMACION') NOT NULL,
    canal ENUM('SMS','EMAIL','PUSH','WHATSAPP') NOT NULL,
    mensaje TEXT NOT NULL,
    estado ENUM('PENDIENTE','ENVIADO','FALLIDO') DEFAULT 'PENDIENTE',
    fecha_programada TIMESTAMP NOT NULL,
    fecha_envio TIMESTAMP NULL,
    respuesta_api TEXT,
    
    FOREIGN KEY (usuario_id) REFERENCES usuarios(usuario_id) ON DELETE CASCADE,
    FOREIGN KEY (cita_id) REFERENCES citas(cita_id) ON DELETE CASCADE,
    
    INDEX idx_estado_programada (estado, fecha_programada),
    INDEX idx_usuario_tipo (usuario_id, tipo)
);

-- Tabla INCIDENCIAS
CREATE TABLE incidencias (
    incidencia_id INT PRIMARY KEY AUTO_INCREMENT,
    usuario_reporta_id INT NOT NULL,
    usuario_asignado_id INT NULL,
    titulo VARCHAR(200) NOT NULL,
    descripcion TEXT NOT NULL,
    prioridad ENUM('BAJA','MEDIA','ALTA','CRITICA') DEFAULT 'MEDIA',
    estado ENUM('ABIERTO','EN_PROCESO','RESUELTO','CERRADO') DEFAULT 'ABIERTO',
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_cierre TIMESTAMP NULL,
    solucion TEXT,
    
    FOREIGN KEY (usuario_reporta_id) REFERENCES usuarios(usuario_id) ON DELETE CASCADE,
    FOREIGN KEY (usuario_asignado_id) REFERENCES usuarios(usuario_id) ON DELETE SET NULL,
    
    INDEX idx_estado_prioridad (estado, prioridad),
    INDEX idx_fecha_creacion (fecha_creacion)
);

-- Tabla ORDENES_MEDICAS
CREATE TABLE ordenes_medicas (
    orden_id INT PRIMARY KEY AUTO_INCREMENT,
    cita_id INT NOT NULL,
    odontologo_id INT NOT NULL,
    tipo_orden ENUM('FARMACIA','LABORATORIO') NOT NULL,
    descripcion TEXT NOT NULL,
    indicaciones TEXT,
    estado ENUM('PENDIENTE','ENVIADO','COMPLETADO') DEFAULT 'PENDIENTE',
    fecha_emision TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (cita_id) REFERENCES citas(cita_id) ON DELETE CASCADE,
    FOREIGN KEY (odontologo_id) REFERENCES odontologos(odontologo_id) ON DELETE CASCADE,
    
    INDEX idx_estado_fecha (estado, fecha_emision)
);

-- Tabla SESIONES
CREATE TABLE sesiones (
    sesion_id INT PRIMARY KEY AUTO_INCREMENT,
    usuario_id INT NOT NULL,
    token_jwt TEXT NOT NULL,
    fecha_inicio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_expiracion TIMESTAMP NOT NULL,
    ip_address VARCHAR(45),
    user_agent TEXT,
    estado ENUM('ACTIVA','EXPIRADA','REVOCADA') DEFAULT 'ACTIVA',
    
    FOREIGN KEY (usuario_id) REFERENCES usuarios(usuario_id) ON DELETE CASCADE,
    INDEX idx_usuario_estado (usuario_id, estado),
    INDEX idx_fecha_expiracion (fecha_expiracion)
);

-- Tabla de AUDITORIA (Registro de cambios críticos)
CREATE TABLE auditoria (
    auditoria_id INT PRIMARY KEY AUTO_INCREMENT,
    tabla_afectada VARCHAR(50) NOT NULL,
    id_registro INT NOT NULL,
    accion ENUM('INSERT','UPDATE','DELETE') NOT NULL,
    usuario_id INT NOT NULL,
    datos_anteriores JSON,
    datos_nuevos JSON,
    fecha_accion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ip_address VARCHAR(45),
    
    FOREIGN KEY (usuario_id) REFERENCES usuarios(usuario_id) ON DELETE CASCADE,
    INDEX idx_tabla_fecha (tabla_afectada, fecha_accion),
    INDEX idx_usuario_fecha (usuario_id, fecha_accion)
);
```

## 5. Poblado Inicial y Datos de Prueba

### Script de Inserción de Datos Iniciales:

```sql
-- Insertar ROLES
INSERT INTO roles (nombre_rol, descripcion) VALUES
('ADMINISTRADOR', 'Acceso completo al sistema'),
('ODONTOLOGO', 'Profesional odontológico con acceso clínico'),
('RECEPCIONISTA', 'Personal de recepción y citas'),
('ASISTENTE_DENTAL', 'Asistente de odontólogo'),
('PACIENTE', 'Usuario paciente del sistema'),
('SOPORTE_TECNICO', 'Personal de soporte técnico');

-- Insertar PERMISOS
INSERT INTO permisos (nombre_permiso, descripcion, modulo) VALUES
('VER_DASHBOARD', 'Ver panel de control', 'DASHBOARD'),
('GESTIONAR_USUARIOS', 'Crear, editar, eliminar usuarios', 'USUARIOS'),
('GESTIONAR_CITAS', 'Programar, modificar, cancelar citas', 'CITAS'),
('VER_HISTORIAL_CLINICO', 'Acceso al historial clínico', 'CLINICO'),
('REGISTRAR_TRATAMIENTO', 'Registrar tratamientos realizados', 'CLINICO'),
('EMITIR_FACTURAS', 'Generar comprobantes de pago', 'FACTURACION'),
('VER_REPORTES', 'Acceso a reportes y estadísticas', 'REPORTES'),
('GESTIONAR_SERVICIOS', 'Administrar servicios ofrecidos', 'SERVICIOS'),
('ENVIAR_NOTIFICACIONES', 'Enviar mensajes y notificaciones', 'NOTIFICACIONES'),
('GESTIONAR_INCIDENCIAS', 'Administrar tickets de soporte', 'SOPORTE');

-- Asignar permisos a roles
INSERT INTO rol_permisos (rol_id, permiso_id) VALUES
-- ADMINISTRADOR (todos los permisos)
(1, 1), (1, 2), (1, 3), (1, 4), (1, 5), (1, 6), (1, 7), (1, 8), (1, 9), (1, 10),
-- ODONTOLOGO
(2, 1), (2, 3), (2, 4), (2, 5), (2, 9),
-- RECEPCIONISTA
(3, 1), (3, 3), (3, 6), (3, 9),
-- ASISTENTE_DENTAL
(4, 1), (4, 3), (4, 4), (4, 9),
-- PACIENTE
(5, 3), (5, 4),
-- SOPORTE_TECNICO
(6, 1), (6, 10);

-- Insertar SERVICIOS
INSERT INTO servicios (codigo_servicio, nombre, descripcion, precio, duracion_minutos) VALUES
('CONS-001', 'Consulta General', 'Consulta odontológica general', 50.00, 30),
('LIMP-001', 'Limpieza Dental', 'Profilaxis y limpieza dental', 80.00, 45),
('EXTR-001', 'Extracción Simple', 'Extracción de pieza dental simple', 120.00, 30),
('ENDO-001', 'Tratamiento de Conducto', 'Endodoncia de pieza dental', 300.00, 90),
('IMPL-001', 'Implante Dental', 'Colocación de implante dental', 1200.00, 120),
('ORTO-001', 'Ortodoncia', 'Tratamiento ortodóntico', 150.00, 60),
('BLAN-001', 'Blanqueamiento', 'Blanqueamiento dental', 200.00, 60);

-- Insertar USUARIOS de ejemplo
INSERT INTO usuarios (dni, nombres, apellidos, fecha_nacimiento, genero, direccion, telefono, email, password_hash) VALUES
('12345678', 'Juan Carlos', 'Pérez García', '1985-05-15', 'M', 'Av. El Sol 123, Puno', '951234567', 'admin@larana.com', '$2b$10$ejemplo_hash_admin'),
('23456789', 'María Elena', 'Rodríguez López', '1990-08-22', 'F', 'Jr. Lima 456, Puno', '962345678', 'maria.rodriguez@larana.com', '$2b$10$ejemplo_hash_doctor'),
('34567890', 'Ana Patricia', 'Mamani Quispe', '1988-03-10', 'F', 'Av. Floral 789, Puno', '973456789', 'ana.mamani@larana.com', '$2b$10$ejemplo_hash_recep'),
('45678901', 'Carlos Antonio', 'Huanca Chura', '1992-11-05', 'M', 'Jr. Deustua 321, Puno', '984567890', 'paciente1@email.com', '$2b$10$ejemplo_hash_paciente');

-- Asignar roles a usuarios
INSERT INTO usuario_roles (usuario_id, rol_id) VALUES
(1, 1), -- Admin
(2, 2), -- Odontólogo
(3, 3), -- Recepcionista
(4, 5); -- Paciente

-- Crear registros específicos por tipo de usuario
INSERT INTO odontologos (usuario_id, numero_colegiatura, especialidad) VALUES
(2, 'COP-12345', 'Odontología General');

INSERT INTO pacientes (usuario_id, numero_historia_clinica, contacto_emergencia, telefono_emergencia) VALUES
(4, 'HC-000001', 'Rosa Chura Mamani', '951987654');

```sql
-- Insertar CITAS de ejemplo
INSERT INTO citas (paciente_id, odontologo_id, servicio_id, fecha_cita, hora_inicio, hora_fin, estado, motivo, registrado_por) VALUES
(1, 1, 1, '2025-07-05', '09:00:00', '09:30:00', 'PROGRAMADA', 'Consulta de rutina', 3),
(1, 1, 2, '2025-07-12', '10:00:00', '10:45:00', 'CONFIRMADA', 'Limpieza dental programada', 3),
(1, 1, 4, '2025-07-20', '14:00:00', '15:30:00', 'PROGRAMADA', 'Tratamiento de conducto molar', 3);

-- Insertar HISTORIALES_CLINICOS de ejemplo
INSERT INTO historiales_clinicos (paciente_id, cita_id, odontologo_id, diagnostico, tratamiento_realizado, observaciones, proxima_cita_recomendada) VALUES
(1, 1, 1, 'Gingivitis leve, caries en molar superior derecho', 'Consulta general, evaluación clínica', 'Paciente presenta buena higiene oral en general. Requiere tratamiento de caries', '2025-07-12');

-- Insertar FACTURAS de ejemplo
INSERT INTO facturas (cita_id, numero_comprobante, tipo_comprobante, subtotal, igv, total, estado_sunat) VALUES
(1, 'B001-00000001', 'BOLETA', 42.37, 7.63, 50.00, 'VALIDADO');

-- Insertar DETALLE_FACTURAS
INSERT INTO detalle_facturas (factura_id, servicio_id, cantidad, precio_unitario, subtotal) VALUES
(1, 1, 1, 42.37, 42.37);

-- Insertar NOTIFICACIONES de ejemplo
INSERT INTO notificaciones (usuario_id, cita_id, tipo, canal, mensaje, fecha_programada) VALUES
(4, 2, 'RECORDATORIO', 'SMS', 'Recordatorio: Tiene una cita programada para mañana 12/07/2025 a las 10:00 AM con Dra. María Rodríguez', '2025-07-11 18:00:00'),
(4, 2, 'RECORDATORIO', 'EMAIL', 'Estimado paciente, le recordamos su cita de limpieza dental programada para el 12/07/2025 a las 10:00 AM', '2025-07-11 18:00:00');
```

## 6. Optimización y Performance

### Índices Adicionales para Mejor Rendimiento:

```sql
-- Índices compuestos para consultas frecuentes
CREATE INDEX idx_citas_fecha_estado ON citas(fecha_cita, estado);
CREATE INDEX idx_citas_odontologo_fecha_hora ON citas(odontologo_id, fecha_cita, hora_inicio);
CREATE INDEX idx_usuarios_estado_tipo ON usuarios(estado);
CREATE INDEX idx_notificaciones_programada_estado ON notificaciones(fecha_programada, estado);
CREATE INDEX idx_historiales_paciente_fecha ON historiales_clinicos(paciente_id, fecha_registro DESC);
CREATE INDEX idx_facturas_fecha_emision ON facturas(fecha_emision DESC);
CREATE INDEX idx_auditoria_tabla_fecha ON auditoria(tabla_afectada, fecha_accion DESC);

-- Índice para búsquedas de texto en diagnósticos
CREATE FULLTEXT INDEX idx_diagnostico_fulltext ON historiales_clinicos(diagnostico, tratamiento_realizado);
```

### Vistas para Consultas Frecuentes:

```sql
-- Vista para información completa de citas
CREATE VIEW v_citas_completas AS
SELECT 
    c.cita_id,
    c.fecha_cita,
    c.hora_inicio,
    c.hora_fin,
    c.estado,
    c.motivo,
    CONCAT(up.nombres, ' ', up.apellidos) AS paciente_nombre,
    up.dni AS paciente_dni,
    up.telefono AS paciente_telefono,
    CONCAT(uo.nombres, ' ', uo.apellidos) AS odontologo_nombre,
    od.numero_colegiatura,
    od.especialidad,
    s.nombre AS servicio_nombre,
    s.precio AS servicio_precio,
    s.duracion_minutos
FROM citas c
INNER JOIN pacientes p ON c.paciente_id = p.paciente_id
INNER JOIN usuarios up ON p.usuario_id = up.usuario_id
INNER JOIN odontologos od ON c.odontologo_id = od.odontologo_id
INNER JOIN usuarios uo ON od.usuario_id = uo.usuario_id
INNER JOIN servicios s ON c.servicio_id = s.servicio_id
WHERE c.estado != 'CANCELADA';

-- Vista para dashboard de odontólogos
CREATE VIEW v_dashboard_odontologo AS
SELECT 
    od.odontologo_id,
    CONCAT(u.nombres, ' ', u.apellidos) AS odontologo_nombre,
    COUNT(CASE WHEN c.fecha_cita = CURDATE() AND c.estado = 'PROGRAMADA' THEN 1 END) AS citas_hoy,
    COUNT(CASE WHEN c.fecha_cita = CURDATE() AND c.estado = 'ATENDIDA' THEN 1 END) AS atendidas_hoy,
    COUNT(CASE WHEN c.fecha_cita BETWEEN CURDATE() AND DATE_ADD(CURDATE(), INTERVAL 7 DAY) THEN 1 END) AS citas_semana,
    COUNT(CASE WHEN c.estado = 'NO_ASISTIO' AND c.fecha_cita >= DATE_SUB(CURDATE(), INTERVAL 30 DAY) THEN 1 END) AS inasistencias_mes
FROM odontologos od
INNER JOIN usuarios u ON od.usuario_id = u.usuario_id
LEFT JOIN citas c ON od.odontologo_id = c.odontologo_id
WHERE u.estado = 'ACTIVO' AND od.estado_colegiatura = 'ACTIVO'
GROUP BY od.odontologo_id, u.nombres, u.apellidos;

-- Vista para historial clínico completo
CREATE VIEW v_historial_completo AS
SELECT 
    hc.historial_id,
    hc.paciente_id,
    CONCAT(up.nombres, ' ', up.apellidos) AS paciente_nombre,
    p.numero_historia_clinica,
    hc.fecha_registro,
    CONCAT(uo.nombres, ' ', uo.apellidos) AS odontologo_nombre,
    s.nombre AS servicio_realizado,
    hc.diagnostico,
    hc.tratamiento_realizado,
    hc.observaciones,
    hc.proxima_cita_recomendada
FROM historiales_clinicos hc
INNER JOIN pacientes p ON hc.paciente_id = p.paciente_id
INNER JOIN usuarios up ON p.usuario_id = up.usuario_id
INNER JOIN odontologos od ON hc.odontologo_id = od.odontologo_id
INNER JOIN usuarios uo ON od.usuario_id = uo.usuario_id
INNER JOIN citas c ON hc.cita_id = c.cita_id
INNER JOIN servicios s ON c.servicio_id = s.servicio_id
ORDER BY hc.fecha_registro DESC;
```

### Triggers para Auditoría Automática:

```sql
-- Trigger para auditar cambios en usuarios
DELIMITER $$

CREATE TRIGGER tr_usuarios_audit_update
AFTER UPDATE ON usuarios
FOR EACH ROW
BEGIN
    INSERT INTO auditoria (
        tabla_afectada, 
        id_registro, 
        accion, 
        usuario_id, 
        datos_anteriores, 
        datos_nuevos,
        ip_address
    ) VALUES (
        'usuarios',
        NEW.usuario_id,
        'UPDATE',
        NEW.usuario_id,
        JSON_OBJECT(
            'dni', OLD.dni,
            'nombres', OLD.nombres,
            'apellidos', OLD.apellidos,
            'email', OLD.email,
            'estado', OLD.estado
        ),
        JSON_OBJECT(
            'dni', NEW.dni,
            'nombres', NEW.nombres,
            'apellidos', NEW.apellidos,
            'email', NEW.email,
            'estado', NEW.estado
        ),
        @client_ip
    );
END$$

-- Trigger para auditar cambios en citas
CREATE TRIGGER tr_citas_audit_update
AFTER UPDATE ON citas
FOR EACH ROW
BEGIN
    INSERT INTO auditoria (
        tabla_afectada, 
        id_registro, 
        accion, 
        usuario_id, 
        datos_anteriores, 
        datos_nuevos,
        ip_address
    ) VALUES (
        'citas',
        NEW.cita_id,
        'UPDATE',
        COALESCE(NEW.registrado_por, 1),
        JSON_OBJECT(
            'fecha_cita', OLD.fecha_cita,
            'hora_inicio', OLD.hora_inicio,
            'estado', OLD.estado,
            'odontologo_id', OLD.odontologo_id
        ),
        JSON_OBJECT(
            'fecha_cita', NEW.fecha_cita,
            'hora_inicio', NEW.hora_inicio,
            'estado', NEW.estado,
            'odontologo_id', NEW.odontologo_id
        ),
        @client_ip
    );
END$$

DELIMITER ;
```

### Procedimientos Almacenados para Operaciones Comunes:

```sql
-- Procedimiento para programar cita con validaciones
DELIMITER $$

CREATE PROCEDURE sp_programar_cita(
    IN p_paciente_id INT,
    IN p_odontologo_id INT,
    IN p_servicio_id INT,
    IN p_fecha_cita DATE,
    IN p_hora_inicio TIME,
    IN p_motivo TEXT,
    IN p_registrado_por INT,
    OUT p_cita_id INT,
    OUT p_mensaje VARCHAR(255)
)
BEGIN
    DECLARE v_hora_fin TIME;
    DECLARE v_duracion INT;
    DECLARE v_conflicto INT DEFAULT 0;
    DECLARE v_horario_inicio TIME;
    DECLARE v_horario_fin TIME;
    
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_mensaje = 'Error al programar la cita';
        SET p_cita_id = 0;
    END;
    
    START TRANSACTION;
    
    -- Obtener duración del servicio
    SELECT duracion_minutos INTO v_duracion
    FROM servicios 
    WHERE servicio_id = p_servicio_id AND estado = 'ACTIVO';
    
    IF v_duracion IS NULL THEN
        SET p_mensaje = 'Servicio no encontrado o inactivo';
        SET p_cita_id = 0;
        ROLLBACK;
    ELSE
        -- Calcular hora de fin
        SET v_hora_fin = ADDTIME(p_hora_inicio, SEC_TO_TIME(v_duracion * 60));
        
        -- Verificar horario del odontólogo
        SELECT horario_inicio, horario_fin INTO v_horario_inicio, v_horario_fin
        FROM odontologos 
        WHERE odontologo_id = p_odontologo_id AND estado_colegiatura = 'ACTIVO';
        
        IF p_hora_inicio < v_horario_inicio OR v_hora_fin > v_horario_fin THEN
            SET p_mensaje = 'Horario fuera del rango de atención del odontólogo';
            SET p_cita_id = 0;
            ROLLBACK;
        ELSE
            -- Verificar conflictos de horario
            SELECT COUNT(*) INTO v_conflicto
            FROM citas 
            WHERE odontologo_id = p_odontologo_id 
            AND fecha_cita = p_fecha_cita
            AND estado NOT IN ('CANCELADA', 'NO_ASISTIO')
            AND (
                (p_hora_inicio >= hora_inicio AND p_hora_inicio < hora_fin) OR
                (v_hora_fin > hora_inicio AND v_hora_fin <= hora_fin) OR
                (p_hora_inicio <= hora_inicio AND v_hora_fin >= hora_fin)
            );
            
            IF v_conflicto > 0 THEN
                SET p_mensaje = 'Conflicto de horario con otra cita';
                SET p_cita_id = 0;
                ROLLBACK;
            ELSE
                -- Insertar la cita
                INSERT INTO citas (
                    paciente_id, odontologo_id, servicio_id, 
                    fecha_cita, hora_inicio, hora_fin, 
                    estado, motivo, registrado_por
                ) VALUES (
                    p_paciente_id, p_odontologo_id, p_servicio_id,
                    p_fecha_cita, p_hora_inicio, v_hora_fin,
                    'PROGRAMADA', p_motivo, p_registrado_por
                );
                
                SET p_cita_id = LAST_INSERT_ID();
                SET p_mensaje = 'Cita programada exitosamente';
                
                COMMIT;
            END IF;
        END IF;
    END IF;
END$$

-- Procedimiento para obtener disponibilidad de odontólogo
CREATE PROCEDURE sp_obtener_disponibilidad(
    IN p_odontologo_id INT,
    IN p_fecha DATE,
    IN p_duracion_minutos INT
)
BEGIN
    SELECT 
        TIME_FORMAT(hora_disponible, '%H:%i') as hora_disponible,
        ADDTIME(hora_disponible, SEC_TO_TIME(p_duracion_minutos * 60)) as hora_fin_estimada
    FROM (
        SELECT 
            ADDTIME(od.horario_inicio, SEC_TO_TIME(slot.minuto * 60)) as hora_disponible
        FROM odontologos od
        CROSS JOIN (
            SELECT 0 as minuto UNION SELECT 30 UNION SELECT 60 UNION SELECT 90 UNION 
            SELECT 120 UNION SELECT 150 UNION SELECT 180 UNION SELECT 210 UNION 
            SELECT 240 UNION SELECT 270 UNION SELECT 300 UNION SELECT 330 UNION 
            SELECT 360 UNION SELECT 390 UNION SELECT 420 UNION SELECT 450 UNION 
            SELECT 480 UNION SELECT 510 UNION SELECT 540 UNION SELECT 570
        ) slot
        WHERE od.odontologo_id = p_odontologo_id
        AND ADDTIME(od.horario_inicio, SEC_TO_TIME((slot.minuto + p_duracion_minutos) * 60)) <= od.horario_fin
    ) horarios_posibles
    WHERE hora_disponible NOT IN (
        SELECT hora_inicio 
        FROM citas 
        WHERE odontologo_id = p_odontologo_id 
        AND fecha_cita = p_fecha 
        AND estado NOT IN ('CANCELADA', 'NO_ASISTIO')
    )
    ORDER BY hora_disponible;
END$$

DELIMITER ;
```

## 7. Documentación Técnica

### Diccionario de Datos

#### Tabla USUARIOS
| Campo | Tipo | Descripción | Restricciones |
|-------|------|-------------|---------------|
| usuario_id | INT | Identificador único del usuario | PK, AUTO_INCREMENT |
| dni | VARCHAR(8) | Documento Nacional de Identidad | NOT NULL, UNIQUE |
| nombres | VARCHAR(100) | Nombres del usuario | NOT NULL |
| apellidos | VARCHAR(100) | Apellidos del usuario | NOT NULL |
| fecha_nacimiento | DATE | Fecha de nacimiento | NOT NULL |
| genero | ENUM('M','F') | Género del usuario | NOT NULL |
| email | VARCHAR(100) | Correo electrónico | UNIQUE |
| password_hash | VARCHAR(255) | Contraseña encriptada | NOT NULL |
| estado | ENUM('ACTIVO','INACTIVO') | Estado del usuario | DEFAULT 'ACTIVO' |

#### Tabla CITAS
| Campo | Tipo | Descripción | Restricciones |
|-------|------|-------------|---------------|
| cita_id | INT | Identificador único de la cita | PK, AUTO_INCREMENT |
| paciente_id | INT | Referencia al paciente | FK -> PACIENTES |
| odontologo_id | INT | Referencia al odontólogo | FK -> ODONTOLOGOS |
| servicio_id | INT | Referencia al servicio | FK -> SERVICIOS |
| fecha_cita | DATE | Fecha de la cita | NOT NULL |
| hora_inicio | TIME | Hora de inicio | NOT NULL |
| hora_fin | TIME | Hora de finalización | NOT NULL |
| estado | ENUM | Estado actual de la cita | Ver valores válidos |

### Reglas de Negocio Implementadas

1. **Gestión de Usuarios:**
   - Cada usuario debe tener un DNI único de 8 dígitos
   - El email debe ser único en el sistema
   - Las contraseñas se almacenan hasheadas con bcrypt

2. **Programación de Citas:**
   - No se pueden programar citas en horarios que se solapen
   - Las citas deben estar dentro del horario de atención del odontólogo
   - La duración de la cita se calcula automáticamente según el servicio

3. **Facturación:**
   - Cada cita atendida debe generar una factura
   - Los comprobantes deben ser validados con SUNAT
   - El IGV se calcula automáticamente (18%)

4. **Notificaciones:**
   - Se envían recordatorios automáticos 24 horas antes de la cita
   - Las notificaciones pueden ser por SMS, email o push
   - Se registra el estado de cada envío

5. **Auditoría:**
   - Todos los cambios críticos se registran automáticamente
   - Se guarda información del usuario que realizó el cambio
   - Se almacena el estado anterior y posterior de los datos

### Consideraciones de Seguridad

1. **Autenticación:**
   - Contraseñas hasheadas con bcrypt (factor 10)
   - Tokens JWT para sesiones con expiración
   - Control de sesiones activas por usuario

2. **Autorización:**
   - Sistema de roles y permisos granular
   - Validación de permisos en cada operación
   - Separación de responsabilidades por tipo de usuario

3. **Integridad de Datos:**
   - Claves foráneas con restricciones ON DELETE
   - Validaciones a nivel de base de datos
   - Triggers para mantener consistencia

4. **Auditoría:**
   - Registro completo de operaciones críticas
   - Trazabilidad de cambios con usuario y timestamp
   - Backup automático de datos sensibles

### Mantenimiento y Respaldos

```sql
-- Script de respaldo diario
-- Ejecutar mediante cron job
mysqldump --routines --triggers --single-transaction larana_clinic_db > backup_$(date +%Y%m%d).sql

-- Limpieza de datos antiguos (ejecutar mensualmente)
DELETE FROM auditoria WHERE fecha_accion < DATE_SUB(NOW(), INTERVAL 1 YEAR);
DELETE FROM sesiones WHERE fecha_expiracion < DATE_SUB(NOW(), INTERVAL 1 MONTH);
DELETE FROM notificaciones WHERE fecha_envio < DATE_SUB(NOW(), INTERVAL 6 MONTH) AND estado = 'ENVIADO';
```

### Consultas de Ejemplo para Validación

```sql
-- Verificar integridad referencial
SELECT 'Pacientes sin usuario' as verificacion, COUNT(*) as registros
FROM pacientes p 
LEFT JOIN usuarios u ON p.usuario_id = u.usuario_id 
WHERE u.usuario_id IS NULL

UNION ALL

SELECT 'Citas sin paciente válido', COUNT(*)
FROM citas c 
LEFT JOIN pacientes p ON c.paciente_id = p.paciente_id 
WHERE p.paciente_id IS NULL

UNION ALL

SELECT 'Facturas sin cita válida', COUNT(*)
FROM facturas f 
LEFT JOIN citas c ON f.cita_id = c.cita_id 
WHERE c.cita_id IS NULL;

-- Reporte de performance
SELECT 
    'Citas programadas hoy' as metrica,
    COUNT(*) as valor
FROM citas 
WHERE fecha_cita = CURDATE() AND estado = 'PROGRAMADA'

UNION ALL

SELECT 'Pacientes atendidos este mes', COUNT(DISTINCT paciente_id)
FROM citas 
WHERE fecha_cita >= DATE_FORMAT(NOW(), '%Y-%m-01') AND estado = 'ATENDIDA'

UNION ALL

SELECT 'Ingresos del mes actual', COALESCE(SUM(total), 0)
FROM facturas 
WHERE fecha_emision >= DATE_FORMAT(NOW(), '%Y-%m-01') AND estado_sunat = 'VALIDADO';
```

Este modelo de base de datos proporciona una base sólida para un sistema integral de gestión de clínica dental, con consideraciones de escalabilidad, seguridad y mantenimiento a largo plazo.
