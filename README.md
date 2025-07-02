# Documentación Completa de Base de Datos
## Sistema de Gestión de Clínica Dental LARANA

### Descripción General del Sistema

El Sistema de Gestión de Clínica Dental LARANA es una aplicación integral que maneja todas las operaciones de una clínica dental moderna. El sistema permite la gestión de usuarios con diferentes roles (administradores, recepcionistas, odontólogos, asistentes y pacientes), programación de citas, manejo de historiales clínicos, facturación electrónica, y comunicaciones automatizadas.

La base de datos está diseñada siguiendo las mejores prácticas de normalización y seguridad, con soporte para auditoría completa, validación con sistemas externos (RENIEC, SUNAT), y arquitectura modular para facilitar el mantenimiento y escalabilidad.

## 1. Documentación Técnica de la Base de Datos

### 1.1 Módulo de Autenticación y Autorización

#### Tabla: `role`
**Propósito**: Gestiona los diferentes roles del sistema con sus respectivas descripciones y estados.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | BIGINT UNSIGNED PK | Identificador único del rol |
| name | VARCHAR(100) UNIQUE | Nombre del rol (admin, receptionist, dentist, assistant, patient) |
| description | TEXT | Descripción detallada del rol |
| is_active | BOOLEAN | Estado activo/inactivo del rol |
| created_at, updated_at | TIMESTAMP | Marcas de tiempo Laravel |

**Relaciones**: 
- Uno a muchos con `user` (un rol puede tener múltiples usuarios)
- Muchos a muchos con `permission` a través de `role_permission`

#### Tabla: `permission`
**Propósito**: Define permisos específicos que pueden ser asignados a roles.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | BIGINT UNSIGNED PK | Identificador único del permiso |
| name | VARCHAR(100) UNIQUE | Nombre del permiso |
| description | TEXT | Descripción del permiso |
| module | VARCHAR(50) | Módulo al que pertenece |

**Relaciones**: Muchos a muchos con `role` a través de `role_permission`

#### Tabla: `user`
**Propósito**: Almacena información base de todos los usuarios del sistema (personal y pacientes).

| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | BIGINT UNSIGNED PK | Identificador único del usuario |
| dni | VARCHAR(8) UNIQUE | Documento Nacional de Identidad |
| email | VARCHAR(191) UNIQUE | Correo electrónico único |
| password | VARCHAR(255) | Contraseña hasheada |
| first_name | VARCHAR(100) | Nombres |
| last_name | VARCHAR(100) | Apellidos |
| phone | VARCHAR(15) | Número de teléfono/celular |
| address | TEXT | Dirección completa |
| birth_date | DATE | Fecha de nacimiento |
| gender | ENUM | Género (male, female, other) |
| role_id | BIGINT UNSIGNED FK | Rol asignado |
| is_active | BOOLEAN | Estado del usuario |
| last_login | TIMESTAMP | Último inicio de sesión |

**Relaciones**: 
- Muchos a uno con `role`
- Uno a uno con `dentist_profile` y `patient_profile`
- Uno a muchos con múltiples tablas como creador/responsable

### 1.2 Módulo de Perfiles Especializados

#### Tabla: `dentist_profile`
**Propósito**: Información específica de los profesionales odontólogos.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| user_id | BIGINT UNSIGNED FK | Referencia al usuario base |
| license_number | VARCHAR(20) UNIQUE | Número de matrícula profesional |
| specialization | VARCHAR(100) | Especialización |
| cop_registration_date | DATE | Fecha de inscripción en COP |
| is_cop_active | BOOLEAN | Estado activo en COP |
| consultation_fee | DECIMAL(8,2) | Tarifa de consulta |

**Pantallas relacionadas**: Panel de Control del Odontólogo, configuración de horarios

#### Tabla: `patient_profile`
**Propósito**: Información específica de los pacientes.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| user_id | BIGINT UNSIGNED FK | Referencia al usuario base |
| emergency_contact_name | VARCHAR(100) | Contacto de emergencia |
| emergency_contact_phone | VARCHAR(15) | Teléfono de emergencia |
| blood_type | VARCHAR(5) | Tipo de sangre |
| allergies | TEXT | Alergias conocidas |
| medical_conditions | TEXT | Condiciones médicas |
| preferred_notification_method | ENUM | Método preferido de notificación |

**Pantallas relacionadas**: Pantalla de Registro de Pacientes, Panel del Recepcionista

### 1.3 Módulo de Gestión de Citas

#### Tabla: `appointment_status`
**Propósito**: Estados posibles de las citas médicas.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | BIGINT UNSIGNED PK | Identificador único |
| name | VARCHAR(50) UNIQUE | Nombre del estado |
| description | TEXT | Descripción del estado |
| color | VARCHAR(7) | Color hexadecimal para UI |

Ejemplos de estados: `scheduled`, `confirmed`, `in_progress`, `completed`, `cancelled`, `no_show`

#### Tabla: `appointment`
**Propósito**: Citas médicas programadas con toda la información necesaria.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | BIGINT UNSIGNED PK | Identificador único |
| patient_id | BIGINT UNSIGNED FK | ID del paciente |
| dentist_id | BIGINT UNSIGNED FK | ID del odontólogo |
| appointment_date | DATE | Fecha de la cita |
| appointment_time | TIME | Hora de la cita |
| duration_minutes | INT | Duración en minutos |
| reason | TEXT | Motivo de la consulta |
| notes | TEXT | Notas adicionales |
| status_id | BIGINT UNSIGNED FK | Estado de la cita |
| created_by | BIGINT UNSIGNED FK | Usuario creador |
| arrived_at | TIMESTAMP | Hora de llegada |
| started_at | TIMESTAMP | Hora de inicio |
| finished_at | TIMESTAMP | Hora de finalización |

**Pantallas relacionadas**: 
- Pantalla de Inicio (agendar cita)
- Panel del Recepcionista (gestión de agenda)
- Panel del Odontólogo (agenda del día)

#### Tabla: `working_hour`
**Propósito**: Horarios de trabajo de los odontólogos por día de la semana.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| dentist_id | BIGINT UNSIGNED FK | ID del odontólogo |
| day_of_week | TINYINT | Día (1=Lunes, 7=Domingo) |
| start_time | TIME | Hora de inicio |
| end_time | TIME | Hora de fin |
| is_active | BOOLEAN | Estado activo |

### 1.4 Módulo de Historial Clínico

#### Tabla: `medical_record`
**Propósito**: Historial clínico completo de cada paciente por visita.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | BIGINT UNSIGNED PK | Identificador único |
| patient_id | BIGINT UNSIGNED FK | ID del paciente |
| dentist_id | BIGINT UNSIGNED FK | ID del odontólogo |
| appointment_id | BIGINT UNSIGNED FK | Cita asociada |
| visit_date | DATE | Fecha de la visita |
| chief_complaint | TEXT | Motivo principal |
| diagnosis | TEXT | Diagnóstico |
| treatment_plan | TEXT | Plan de tratamiento |
| treatment_performed | TEXT | Tratamiento realizado |
| observations | TEXT | Observaciones |
| next_visit_date | DATE | Próxima visita sugerida |

**Pantallas relacionadas**: Panel del Odontólogo (consulta y registro de tratamientos)

#### Tabla: `medical_record_file`
**Propósito**: Archivos adjuntos al historial (rayos X, fotografías, documentos).

| Campo | Tipo | Descripción |
|-------|------|-------------|
| medical_record_id | BIGINT UNSIGNED FK | Referencia al historial |
| file_name | VARCHAR(255) | Nombre original |
| file_path | VARCHAR(500) | Ruta de almacenamiento |
| file_type | VARCHAR(50) | Tipo de archivo |
| file_size | INT | Tamaño en bytes |
| description | TEXT | Descripción del archivo |

### 1.5 Módulo de Facturación

#### Tabla: `service`
**Propósito**: Servicios odontológicos ofrecidos por la clínica.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | BIGINT UNSIGNED PK | Identificador único |
| name | VARCHAR(100) | Nombre del servicio |
| description | TEXT | Descripción detallada |
| price | DECIMAL(8,2) | Precio base |
| duration_minutes | INT | Duración estimada |
| is_active | BOOLEAN | Estado del servicio |

#### Tabla: `invoice`
**Propósito**: Facturas y boletas electrónicas.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | BIGINT UNSIGNED PK | Identificador único |
| invoice_number | VARCHAR(20) UNIQUE | Número de comprobante |
| invoice_type | ENUM | Tipo (boleta, factura) |
| patient_id | BIGINT UNSIGNED FK | ID del paciente |
| appointment_id | BIGINT UNSIGNED FK | Cita asociada |
| issue_date | DATE | Fecha de emisión |
| due_date | DATE | Fecha de vencimiento |
| subtotal | DECIMAL(10,2) | Subtotal sin impuestos |
| tax_amount | DECIMAL(10,2) | Monto de impuestos |
| total_amount | DECIMAL(10,2) | Total a pagar |
| payment_status | ENUM | Estado de pago |
| payment_method | VARCHAR(50) | Método de pago |
| payment_date | DATE | Fecha de pago |
| sunat_status | ENUM | Estado SUNAT |
| sunat_response | TEXT | Respuesta SUNAT |

**Pantallas relacionadas**: Panel del Recepcionista (facturación)

### 1.6 Módulos Auxiliares

#### Tabla: `notification`
**Propósito**: Sistema de notificaciones multi-canal.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| user_id | BIGINT UNSIGNED FK | Usuario destinatario |
| title | VARCHAR(255) | Título |
| message | TEXT | Mensaje |
| type | ENUM | Tipo (info, warning, error, success) |
| method | ENUM | Método (email, sms, whatsapp, system) |
| is_sent | BOOLEAN | Estado de envío |
| is_read | BOOLEAN | Estado de lectura |

#### Tabla: `audit_log`
**Propósito**: Registro completo de auditoría del sistema.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| user_id | BIGINT UNSIGNED FK | Usuario que realizó la acción |
| action | VARCHAR(50) | Acción (create, update, delete) |
| table_name | VARCHAR(50) | Tabla afectada |
| record_id | BIGINT UNSIGNED | ID del registro |
| old_values | JSON | Valores anteriores |
| new_values | JSON | Valores nuevos |
| ip_address | VARCHAR(45) | IP del usuario |
| user_agent | TEXT | User agent |

## 2. Consultas SQL de Prueba

### 2.1 Módulo de Autenticación

```sql
-- Obtener usuarios activos con sus roles
SELECT 
    u.id,
    u.dni,
    u.email,
    CONCAT(u.first_name, ' ', u.last_name) as full_name,
    r.name as role_name,
    u.last_login
FROM user u
INNER JOIN role r ON u.role_id = r.id
WHERE u.is_active = TRUE
ORDER BY u.last_login DESC;

-- Verificar permisos de un usuario específico
SELECT 
    u.email,
    r.name as role_name,
    p.name as permission_name,
    p.module
FROM user u
INNER JOIN role r ON u.role_id = r.id
INNER JOIN role_permission rp ON r.id = rp.role_id
INNER JOIN permission p ON rp.permission_id = p.id
WHERE u.dni = '12345678';
```

### 2.2 Módulo de Citas

```sql
-- Agenda diaria de un odontólogo específico
SELECT 
    a.id,
    a.appointment_date,
    a.appointment_time,
    a.duration_minutes,
    CONCAT(p.first_name, ' ', p.last_name) as patient_name,
    p.phone as patient_phone,
    a.reason,
    s.name as status_name,
    s.color as status_color
FROM appointment a
INNER JOIN user p ON a.patient_id = p.id
INNER JOIN user d ON a.dentist_id = d.id
INNER JOIN appointment_status s ON a.status_id = s.id
WHERE d.dni = '87654321' 
    AND a.appointment_date = CURDATE()
ORDER BY a.appointment_time;

-- Verificar disponibilidad de horario
SELECT 
    wh.day_of_week,
    wh.start_time,
    wh.end_time,
    COUNT(a.id) as appointments_count
FROM working_hour wh
LEFT JOIN appointment a ON a.dentist_id = wh.dentist_id 
    AND DAYOFWEEK(a.appointment_date) = wh.day_of_week + 1
    AND a.appointment_time >= wh.start_time 
    AND a.appointment_time < wh.end_time
    AND a.appointment_date = '2024-07-15'
WHERE wh.dentist_id = (SELECT id FROM user WHERE dni = '87654321')
    AND wh.is_active = TRUE
GROUP BY wh.day_of_week, wh.start_time, wh.end_time;
```

### 2.3 Módulo de Historial Clínico

```sql
-- Historial completo de un paciente
SELECT 
    mr.id,
    mr.visit_date,
    CONCAT(d.first_name, ' ', d.last_name) as dentist_name,
    mr.chief_complaint,
    mr.diagnosis,
    mr.treatment_performed,
    mr.observations,
    mr.next_visit_date
FROM medical_record mr
INNER JOIN user d ON mr.dentist_id = d.id
INNER JOIN user p ON mr.patient_id = p.id
WHERE p.dni = '11223344'
ORDER BY mr.visit_date DESC;

-- Archivos adjuntos de un registro médico
SELECT 
    mrf.file_name,
    mrf.file_type,
    mrf.file_size,
    mrf.description,
    mrf.created_at
FROM medical_record_file mrf
INNER JOIN medical_record mr ON mrf.medical_record_id = mr.id
WHERE mr.id = 1;
```

### 2.4 Módulo de Facturación

```sql
-- Reporte de facturación mensual
SELECT 
    DATE(i.issue_date) as fecha,
    COUNT(*) as total_comprobantes,
    SUM(CASE WHEN i.invoice_type = 'boleta' THEN 1 ELSE 0 END) as boletas,
    SUM(CASE WHEN i.invoice_type = 'factura' THEN 1 ELSE 0 END) as facturas,
    SUM(i.total_amount) as total_facturado,
    SUM(CASE WHEN i.payment_status = 'paid' THEN i.total_amount ELSE 0 END) as total_cobrado
FROM invoice i
WHERE i.issue_date >= '2024-07-01' 
    AND i.issue_date < '2024-08-01'
GROUP BY DATE(i.issue_date)
ORDER BY DATE(i.issue_date);

-- Detalle de factura con servicios
SELECT 
    i.invoice_number,
    i.invoice_type,
    CONCAT(p.first_name, ' ', p.last_name) as patient_name,
    s.name as service_name,
    id_detail.quantity,
    id_detail.unit_price,
    id_detail.total_price
FROM invoice i
INNER JOIN user p ON i.patient_id = p.id
INNER JOIN invoice_detail id_detail ON i.id = id_detail.invoice_id
INNER JOIN service s ON id_detail.service_id = s.id
WHERE i.invoice_number = 'B001-00000001';
```

### 2.5 Ejemplos de INSERT

```sql
-- Insertar nuevo paciente
INSERT INTO user (dni, email, password, first_name, last_name, phone, address, birth_date, gender, role_id, is_active, created_at, updated_at)
VALUES (
    '12345678', 
    'paciente@email.com', 
    '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', -- password: 'password'
    'Juan Carlos', 
    'Pérez Mamani', 
    '987654321', 
    'Av. El Sol 123, Puno', 
    '1985-03-15', 
    'male', 
    (SELECT id FROM role WHERE name = 'patient'), 
    TRUE, 
    NOW(), 
    NOW()
);

-- Insertar perfil de paciente
INSERT INTO patient_profile (user_id, emergency_contact_name, emergency_contact_phone, blood_type, allergies, preferred_notification_method, created_at, updated_at)
VALUES (
    (SELECT id FROM user WHERE dni = '12345678'),
    'María Pérez',
    '987123456',
    'O+',
    'Penicilina',
    'whatsapp',
    NOW(),
    NOW()
);

-- Programar nueva cita
INSERT INTO appointment (patient_id, dentist_id, appointment_date, appointment_time, duration_minutes, reason, status_id, created_by, created_at, updated_at)
VALUES (
    (SELECT id FROM user WHERE dni = '12345678'),
    (SELECT id FROM user WHERE dni = '87654321'),
    '2024-07-15',
    '10:00:00',
    30,
    'Consulta de rutina y limpieza dental',
    (SELECT id FROM appointment_status WHERE name = 'scheduled'),
    (SELECT id FROM user WHERE dni = '11111111'), -- recepcionista
    NOW(),
    NOW()
);
```

### 2.6 Ejemplos de UPDATE y DELETE

```sql
-- Actualizar estado de cita cuando llega el paciente
UPDATE appointment 
SET 
    status_id = (SELECT id FROM appointment_status WHERE name = 'confirmed'),
    arrived_at = NOW(),
    updated_at = NOW()
WHERE id = 1 AND patient_id = (SELECT id FROM user WHERE dni = '12345678');

-- Cancelar cita (soft delete - cambio de estado)
UPDATE appointment 
SET 
    status_id = (SELECT id FROM appointment_status WHERE name = 'cancelled'),
    notes = CONCAT(COALESCE(notes, ''), ' - Cancelada por el paciente el ', NOW()),
    updated_at = NOW()
WHERE id = 1;

-- Desactivar usuario en lugar de eliminarlo
UPDATE user 
SET 
    is_active = FALSE,
    updated_at = NOW()
WHERE dni = '12345678';

-- Eliminar archivo de historial médico (hard delete necesario)
DELETE FROM medical_record_file 
WHERE id = 1 
    AND medical_record_id IN (
        SELECT id FROM medical_record 
        WHERE patient_id = (SELECT id FROM user WHERE dni = '12345678')
    );
```

## 3. Procedimientos Almacenados (Stored Procedures)

### 3.1 Gestión de Usuarios

```sql
DELIMITER //

-- Procedimiento para crear usuario completo con perfil
CREATE PROCEDURE sp_create_patient_user(
    IN p_dni VARCHAR(8),
    IN p_email VARCHAR(191),
    IN p_password VARCHAR(255),
    IN p_first_name VARCHAR(100),
    IN p_last_name VARCHAR(100),
    IN p_phone VARCHAR(15),
    IN p_address TEXT,
    IN p_birth_date DATE,
    IN p_gender ENUM('male', 'female', 'other'),
    IN p_emergency_contact_name VARCHAR(100),
    IN p_emergency_contact_phone VARCHAR(15),
    IN p_blood_type VARCHAR(5),
    IN p_allergies TEXT,
    OUT p_user_id BIGINT,
    OUT p_result_message VARCHAR(255)
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_result_message = 'Error: No se pudo crear el usuario';
        SET p_user_id = 0;
    END;
    
    START TRANSACTION;
    
    -- Verificar si el DNI ya existe
    IF EXISTS(SELECT 1 FROM user WHERE dni = p_dni) THEN
        SET p_result_message = 'Error: DNI ya registrado';
        SET p_user_id = 0;
        ROLLBACK;
    ELSE
        -- Insertar usuario
        INSERT INTO user (dni, email, password, first_name, last_name, phone, address, birth_date, gender, role_id, is_active, created_at, updated_at)
        VALUES (p_dni, p_email, p_password, p_first_name, p_last_name, p_phone, p_address, p_birth_date, p_gender, 
                (SELECT id FROM role WHERE name = 'patient'), TRUE, NOW(), NOW());
        
        SET p_user_id = LAST_INSERT_ID();
        
        -- Insertar perfil de paciente
        INSERT INTO patient_profile (user_id, emergency_contact_name, emergency_contact_phone, blood_type, allergies, preferred_notification_method, created_at, updated_at)
        VALUES (p_user_id, p_emergency_contact_name, p_emergency_contact_phone, p_blood_type, p_allergies, 'email', NOW(), NOW());
        
        SET p_result_message = 'Usuario creado exitosamente';
        COMMIT;
    END IF;
END//

DELIMITER ;
```

### 3.2 Gestión de Citas

```sql
DELIMITER //

-- Procedimiento para agendar cita con validaciones
CREATE PROCEDURE sp_schedule_appointment(
    IN p_patient_dni VARCHAR(8),
    IN p_dentist_dni VARCHAR(8),
    IN p_appointment_date DATE,
    IN p_appointment_time TIME,
    IN p_duration_minutes INT,
    IN p_reason TEXT,
    IN p_created_by_dni VARCHAR(8),
    OUT p_appointment_id BIGINT,
    OUT p_result_message VARCHAR(255)
)
BEGIN
    DECLARE v_patient_id BIGINT;
    DECLARE v_dentist_id BIGINT;
    DECLARE v_created_by_id BIGINT;
    DECLARE v_conflict_count INT DEFAULT 0;
    DECLARE v_working_day_count INT DEFAULT 0;
    
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_result_message = 'Error: No se pudo agendar la cita';
        SET p_appointment_id = 0;
    END;
    
    START TRANSACTION;
    
    -- Obtener IDs de usuarios
    SELECT id INTO v_patient_id FROM user WHERE dni = p_patient_dni AND role_id = (SELECT id FROM role WHERE name = 'patient');
    SELECT id INTO v_dentist_id FROM user WHERE dni = p_dentist_dni AND role_id = (SELECT id FROM role WHERE name = 'dentist');
    SELECT id INTO v_created_by_id FROM user WHERE dni = p_created_by_dni;
    
    -- Validar que los usuarios existan
    IF v_patient_id IS NULL THEN
        SET p_result_message = 'Error: Paciente no encontrado';
        SET p_appointment_id = 0;
        ROLLBACK;
    ELSEIF v_dentist_id IS NULL THEN
        SET p_result_message = 'Error: Odontólogo no encontrado';
        SET p_appointment_id = 0;
        ROLLBACK;
    ELSE
        -- Verificar conflictos de horario
        SELECT COUNT(*) INTO v_conflict_count
        FROM appointment
        WHERE dentist_id = v_dentist_id
            AND appointment_date = p_appointment_date
            AND appointment_time = p_appointment_time
            AND status_id NOT IN (SELECT id FROM appointment_status WHERE name IN ('cancelled', 'no_show'));
        
        -- Verificar horario laboral
        SELECT COUNT(*) INTO v_working_day_count
        FROM working_hour
        WHERE dentist_id = v_dentist_id
            AND day_of_week = DAYOFWEEK(p_appointment_date) - 1
            AND p_appointment_time >= start_time
            AND p_appointment_time < end_time
            AND is_active = TRUE;
        
        IF v_conflict_count > 0 THEN
            SET p_result_message = 'Error: Horario no disponible';
            SET p_appointment_id = 0;
            ROLLBACK;
        ELSEIF v_working_day_count = 0 THEN
            SET p_result_message = 'Error: Fuera del horario laboral';
            SET p_appointment_id = 0;
            ROLLBACK;
        ELSE
            -- Crear la cita
            INSERT INTO appointment (patient_id, dentist_id, appointment_date, appointment_time, duration_minutes, reason, 
                                   status_id, created_by, created_at, updated_at)
            VALUES (v_patient_id, v_dentist_id, p_appointment_date, p_appointment_time, p_duration_minutes, p_reason,
                    (SELECT id FROM appointment_status WHERE name = 'scheduled'), v_created_by_id, NOW(), NOW());
            
            SET p_appointment_id = LAST_INSERT_ID();
            SET p_result_message = 'Cita agendada exitosamente';
            COMMIT;
        END IF;
    END IF;
END//

DELIMITER ;
```

### 3.3 Gestión de Historial Clínico

```sql
DELIMITER //

-- Procedimiento para registrar consulta médica
CREATE PROCEDURE sp_create_medical_record(
    IN p_patient_dni VARCHAR(8),
    IN p_dentist_dni VARCHAR(8),
    IN p_appointment_id BIGINT,
    IN p_chief_complaint TEXT,
    IN p_diagnosis TEXT,
    IN p_treatment_plan TEXT,
    IN p_treatment_performed TEXT,
    IN p_observations TEXT,
    IN p_next_visit_date DATE,
    OUT p_record_id BIGINT,
    OUT p_result_message VARCHAR(255)
)
BEGIN
    DECLARE v_patient_id BIGINT;
    DECLARE v_dentist_id BIGINT;
    
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_result_message = 'Error: No se pudo crear el registro médico';
        SET p_record_id = 0;
    END;
    
    START TRANSACTION;
    
    -- Obtener IDs
    SELECT id INTO v_patient_id FROM user WHERE dni = p_patient_dni;
    SELECT id INTO v_dentist_id FROM user WHERE dni = p_dentist_dni;
    
    IF v_patient_id IS NULL OR v_dentist_id IS NULL THEN
        SET p_result_message = 'Error: Usuario no encontrado';
        SET p_record_id = 0;
        ROLLBACK;
    ELSE
        -- Crear registro médico
        INSERT INTO medical_record (patient_id, dentist_id, appointment_id, visit_date, chief_complaint, 
                                  diagnosis, treatment_plan, treatment_performed, observations, next_visit_date, 
                                  created_at, updated_at)
        VALUES (v_patient_id, v_dentist_id, p_appointment_id, CURDATE(), p_chief_complaint, 
                p_diagnosis, p_treatment_plan, p_treatment_performed, p_observations, p_next_visit_date, NOW(), NOW());
        
        SET p_record_id = LAST_INSERT_ID();
        
        -- Actualizar estado de la cita si existe
        IF p_appointment_id IS NOT NULL THEN
            UPDATE appointment 
            SET status_id = (SELECT id FROM appointment_status WHERE name = 'completed'),
                finished_at = NOW(),
                updated_at = NOW()
            WHERE id = p_appointment_id;
        END IF;
        
        SET p_result_message = 'Registro médico creado exitosamente';
        COMMIT;
    END IF;
END//

DELIMITER ;
```

### 3.4 Gestión de Facturación

```sql
DELIMITER //

-- Procedimiento para generar factura
CREATE PROCEDURE sp_generate_invoice(
    IN p_patient_dni VARCHAR(8),
    IN p_appointment_id BIGINT,
    IN p_invoice_type ENUM('boleta', 'factura'),
    IN p_services_json JSON, -- [{"service_id": 1, "quantity": 1}, ...]
    IN p_created_by_dni VARCHAR(8),
    OUT p_invoice_id BIGINT,
    OUT p_invoice_number VARCHAR(20),
    OUT p_result_message VARCHAR(255)
)
BEGIN
    DECLARE v_next_number INT;
    DECLARE v_service_count INT DEFAULT 0;
    DECLARE v_i INT DEFAULT 0;
    DECLARE v_service_id BIGINT;
    DECLARE v_quantity INT;
    DECLARE v_service_price DECIMAL(8,2);
    DECLARE v_line_total DECIMAL(10,2);
    
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_result_message = 'Error: No se pudo generar la factura';
        SET p_invoice_id = 0;
        SET p_invoice_number = NULL;
    END;
    
    START TRANSACTION;
    
    -- Obtener IDs de usuarios
    SELECT id INTO v_patient_id FROM user WHERE dni = p_patient_dni;
    SELECT id INTO v_created_by_id FROM user WHERE dni = p_created_by_dni;
    
    IF v_patient_id IS NULL OR v_created_by_id IS NULL THEN
        SET p_result_message = 'Error: Usuario no encontrado';
        SET p_invoice_id = 0;
        SET p_invoice_number = NULL;
        ROLLBACK;
    ELSE
        -- Generar número de factura
        SELECT COALESCE(MAX(CAST(SUBSTRING(invoice_number, -8) AS UNSIGNED)), 0) + 1 
        INTO v_next_number 
        FROM invoice 
        WHERE invoice_type = p_invoice_type;
        
        SET p_invoice_number = CONCAT(
            IF(p_invoice_type = 'boleta', 'B001-', 'F001-'),
            LPAD(v_next_number, 8, '0')
        );
        
        -- Calcular totales de servicios
        SET v_service_count = JSON_LENGTH(p_services_json);
        SET v_i = 0;
        
        WHILE v_i < v_service_count DO
            SET v_service_id = JSON_UNQUOTE(JSON_EXTRACT(p_services_json, CONCAT('$[', v_i, '].service_id')));
            SET v_quantity = JSON_UNQUOTE(JSON_EXTRACT(p_services_json, CONCAT('$[', v_i, '].quantity')));
            
            SELECT price INTO v_service_price FROM service WHERE id = v_service_id;
            SET v_line_total = v_service_price * v_quantity;
            SET v_subtotal = v_subtotal + v_line_total;
            
            SET v_i = v_i + 1;
        END WHILE;
        
        -- Calcular impuestos (IGV 18%)
        SET v_tax_amount = ROUND(v_subtotal * 0.18, 2);
        SET v_total_amount = v_subtotal + v_tax_amount;
        
        -- Crear factura
        INSERT INTO invoice (invoice_number, invoice_type, patient_id, appointment_id, issue_date, 
                           due_date, subtotal, tax_amount, total_amount, payment_status, 
                           created_by, created_at, updated_at)
        VALUES (p_invoice_number, p_invoice_type, v_patient_id, p_appointment_id, CURDATE(),
                DATE_ADD(CURDATE(), INTERVAL 30 DAY), v_subtotal, v_tax_amount, v_total_amount,
                'pending', v_created_by_id, NOW(), NOW());
        
        SET p_invoice_id = LAST_INSERT_ID();
        
        -- Insertar detalles de factura
        SET v_i = 0;
        WHILE v_i < v_service_count DO
            SET v_service_id = JSON_UNQUOTE(JSON_EXTRACT(p_services_json, CONCAT('$[', v_i, '].service_id')));
            SET v_quantity = JSON_UNQUOTE(JSON_EXTRACT(p_services_json, CONCAT('$[', v_i, '].quantity')));
            
            SELECT price INTO v_service_price FROM service WHERE id = v_service_id;
            SET v_line_total = v_service_price * v_quantity;
            
            INSERT INTO invoice_detail (invoice_id, service_id, quantity, unit_price, total_price, created_at, updated_at)
            VALUES (p_invoice_id, v_service_id, v_quantity, v_service_price, v_line_total, NOW(), NOW());
            
            SET v_i = v_i + 1;
        END WHILE;
        
        SET p_result_message = 'Factura generada exitosamente';
        COMMIT;
    END IF;
END//

DELIMITER ;

```

## 4. Triggers del Sistema

### 4.1 Trigger de Auditoría para Usuario

```sql
DELIMITER //

-- Trigger para auditar cambios en la tabla user
CREATE TRIGGER tr_user_audit_update
AFTER UPDATE ON user
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (user_id, action, table_name, record_id, old_values, new_values, ip_address, created_at)
    VALUES (
        NEW.id,
        'update',
        'user',
        NEW.id,
        JSON_OBJECT(
            'dni', OLD.dni,
            'email', OLD.email,
            'first_name', OLD.first_name,
            'last_name', OLD.last_name,
            'phone', OLD.phone,
            'address', OLD.address,
            'is_active', OLD.is_active,
            'role_id', OLD.role_id
        ),
        JSON_OBJECT(
            'dni', NEW.dni,
            'email', NEW.email,
            'first_name', NEW.first_name,
            'last_name', NEW.last_name,
            'phone', NEW.phone,
            'address', NEW.address,
            'is_active', NEW.is_active,
            'role_id', NEW.role_id
        ),
        @user_ip_address,
        NOW()
    );
END//

DELIMITER ;
```

### 4.2 Trigger de Auditoría para Citas

```sql
DELIMITER //

-- Trigger para auditar cambios de estado en citas
CREATE TRIGGER tr_appointment_status_audit
AFTER UPDATE ON appointment
FOR EACH ROW
BEGIN
    IF OLD.status_id != NEW.status_id THEN
        INSERT INTO audit_log (user_id, action, table_name, record_id, old_values, new_values, created_at)
        VALUES (
            @current_user_id,
            'status_change',
            'appointment',
            NEW.id,
            JSON_OBJECT('status_id', OLD.status_id, 'notes', OLD.notes),
            JSON_OBJECT('status_id', NEW.status_id, 'notes', NEW.notes),
            NOW()
        );
    END IF;
END//

DELIMITER ;
```

### 4.3 Trigger de Notificación para Citas

```sql
DELIMITER //

-- Trigger para enviar notificaciones cuando se programa una cita
CREATE TRIGGER tr_appointment_notification
AFTER INSERT ON appointment
FOR EACH ROW
BEGIN
    DECLARE v_patient_name VARCHAR(201);
    DECLARE v_dentist_name VARCHAR(201);
    DECLARE v_notification_method ENUM('email', 'sms', 'whatsapp');
    
    -- Obtener nombres y método de notificación preferido
    SELECT 
        CONCAT(u.first_name, ' ', u.last_name),
        pp.preferred_notification_method
    INTO v_patient_name, v_notification_method
    FROM user u
    LEFT JOIN patient_profile pp ON u.id = pp.user_id
    WHERE u.id = NEW.patient_id;
    
    SELECT CONCAT(first_name, ' ', last_name) 
    INTO v_dentist_name 
    FROM user 
    WHERE id = NEW.dentist_id;
    
    -- Crear notificación para el paciente
    INSERT INTO notification (user_id, title, message, type, method, created_at, updated_at)
    VALUES (
        NEW.patient_id,
        'Cita Programada',
        CONCAT('Su cita con ', v_dentist_name, ' ha sido programada para el ', 
               DATE_FORMAT(NEW.appointment_date, '%d/%m/%Y'), ' a las ', 
               TIME_FORMAT(NEW.appointment_time, '%H:%i')),
        'info',
        COALESCE(v_notification_method, 'email'),
        NOW(),
        NOW()
    );
    
    -- Crear notificación para el odontólogo
    INSERT INTO notification (user_id, title, message, type, method, created_at, updated_at)
    VALUES (
        NEW.dentist_id,
        'Nueva Cita Programada',
        CONCAT('Nueva cita con ', v_patient_name, ' programada para el ', 
               DATE_FORMAT(NEW.appointment_date, '%d/%m/%Y'), ' a las ', 
               TIME_FORMAT(NEW.appointment_time, '%H:%i')),
        'info',
        'system',
        NOW(),
        NOW()
    );
END//

DELIMITER ;
```

### 4.4 Trigger de Validación de Horarios

```sql
DELIMITER //

-- Trigger para validar horarios antes de insertar cita
CREATE TRIGGER tr_appointment_validate_schedule
BEFORE INSERT ON appointment
FOR EACH ROW
BEGIN
    DECLARE v_conflict_count INT DEFAULT 0;
    DECLARE v_working_hours_count INT DEFAULT 0;
    
    -- Verificar conflictos de horario
    SELECT COUNT(*) INTO v_conflict_count
    FROM appointment
    WHERE dentist_id = NEW.dentist_id
        AND appointment_date = NEW.appointment_date
        AND appointment_time = NEW.appointment_time
        AND status_id NOT IN (SELECT id FROM appointment_status WHERE name IN ('cancelled', 'no_show'));
    
    IF v_conflict_count > 0 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Horario no disponible - Ya existe una cita programada';
    END IF;
    
    -- Verificar horario laboral
    SELECT COUNT(*) INTO v_working_hours_count
    FROM working_hour
    WHERE dentist_id = NEW.dentist_id
        AND day_of_week = DAYOFWEEK(NEW.appointment_date) - 1
        AND NEW.appointment_time >= start_time
        AND NEW.appointment_time < end_time
        AND is_active = TRUE;
    
    IF v_working_hours_count = 0 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Horario fuera del horario laboral del odontólogo';
    END IF;
END//

DELIMITER ;
```

## 5. Funciones Definidas por el Usuario (UDFs)

### 5.1 Función para Calcular Edad

```sql
DELIMITER //

-- Función para calcular la edad de un paciente
CREATE FUNCTION fn_calculate_age(birth_date DATE)
RETURNS INT
READS SQL DATA
DETERMINISTIC
BEGIN
    DECLARE age INT;
    
    IF birth_date IS NULL THEN
        RETURN NULL;
    END IF;
    
    SET age = TIMESTAMPDIFF(YEAR, birth_date, CURDATE());
    
    RETURN age;
END//

DELIMITER ;
```

### 5.2 Función para Validar DNI

```sql
DELIMITER //

-- Función para validar formato de DNI peruano
CREATE FUNCTION fn_validate_dni(dni_number VARCHAR(8))
RETURNS BOOLEAN
READS SQL DATA
DETERMINISTIC
BEGIN
    -- Verificar que tenga exactamente 8 dígitos
    IF LENGTH(dni_number) != 8 THEN
        RETURN FALSE;
    END IF;
    
    -- Verificar que solo contenga números
    IF dni_number REGEXP '^[0-9]{8}    DECLARE v_next_number INT;
    DECLARE v_service_count INT DEFAULT 0;
    DECLARE v_i INT DEFAULT 0;
    DECLARE v_service_id BIGINT;
    DECLARE v_quantity INT;
    DECLARE v_service_price DECIMAL(8,2);
    DECLARE v_line_total DECIMAL(10,2);
    
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_result_message = 'Error: No se pudo generar la factura';
        SET p_invoice_id = 0;
        SET p_invoice_number = NULL;
    END;
    
    START TRANSACTION;
    
    -- Obtener IDs de usuarios
    SELECT id INTO v_patient_id FROM user WHERE dni = p_patient_dni;
    SELECT id INTO v_created_by_id FROM user WHERE dni = p_created_by_dni;
    
    IF v_patient_id IS NULL OR v_created_by_id IS NULL THEN
        SET p_result_message = 'Error: Usuario no encontrado';
        SET p_invoice_id = 0;
        SET p_invoice_number = NULL;
        ROLLBACK;
    ELSE
        -- Generar número de factura
        SELECT COALESCE(MAX(CAST(SUBSTRING(invoice_number, -8) AS UNSIGNED)), 0) + 1 
        INTO v_next_number 
        FROM invoice 
        WHERE invoice_type = p_invoice_type;
        
        SET p_invoice_number = CONCAT(
            IF(p_invoice_type = 'boleta', 'B001-', 'F001-'),
            LPAD(v_next_number, 8, '0')
        );
        
        -- Calcular totales de servicios
        SET v_service_count = JSON_LENGTH(p_services_json);
        SET v_i = 0;
        
        WHILE v_i < v_service_count DO
            SET v_service_id = JSON_UNQUOTE(JSON_EXTRACT(p_services_json, CONCAT('$[', v_i, '].service_id')));
            SET v_quantity = JSON_UNQUOTE(JSON_EXTRACT(p_services_json, CONCAT('$[', v_i, '].quantity')));
            
            SELECT price INTO v_service_price FROM service WHERE id = v_service_id;
            SET v_line_total = v_service_price * v_quantity;
            SET v_subtotal = v_subtotal + v_line_total;
            
            SET v_i = v_i + 1;
        END WHILE;
        
        -- Calcular impuestos (IGV 18%)
        SET v_tax_amount = ROUND(v_subtotal * 0.18, 2);
        SET v_total_amount = v_subtotal + v_tax_amount;
        
        -- Crear factura
        INSERT INTO invoice (invoice_number, invoice_type, patient_id, appointment_id, issue_date, 
                           due_date, subtotal, tax_amount, total_amount, payment_status, 
                           created_by, created_at, updated_at)
        VALUES (p_invoice_number, p_invoice_type, v_patient_id, p_appointment_id, CURDATE(),
                DATE_ADD(CURDATE(), INTERVAL 30 DAY), v_subtotal, v_tax_amount, v_total_amount,
                'pending', v_created_by_id, NOW(), NOW());
        
        SET p_invoice_id = LAST_INSERT_ID();
        
        -- Insertar detalles de factura
        SET v_i = 0;
        WHILE v_i < v_service_count DO
            SET v_service_id = JSON_UNQUOTE(JSON_EXTRACT(p_services_json, CONCAT('$[', v_i, '].service_id')));
            SET v_quantity = JSON_UNQUOTE(JSON_EXTRACT(p_services_json, CONCAT('$[', v_i, '].quantity')));
            
            SELECT price INTO v_service_price FROM service WHERE id = v_service_id;
            SET v_line_total = v_service_price * v_quantity;
            
            INSERT INTO invoice_detail (invoice_id, service_id, quantity, unit_price, total_price, created_at, updated_at)
            VALUES (p_invoice_id, v_service_id, v_quantity, v_service_price, v_line_total, NOW(), NOW());
            
            SET v_i = v_i + 1;
        END WHILE;
        
        SET p_result_message = 'Factura generada exitosamente';
        COMMIT;
    END IF;
END//

DELIMITER ; = 0 THEN
        RETURN FALSE;
    END IF;
    
    -- Verificar que no sean todos números iguales
    IF dni_number REGEXP '^([0-9])\\1{7}    DECLARE v_next_number INT;
    DECLARE v_service_count INT DEFAULT 0;
    DECLARE v_i INT DEFAULT 0;
    DECLARE v_service_id BIGINT;
    DECLARE v_quantity INT;
    DECLARE v_service_price DECIMAL(8,2);
    DECLARE v_line_total DECIMAL(10,2);
    
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_result_message = 'Error: No se pudo generar la factura';
        SET p_invoice_id = 0;
        SET p_invoice_number = NULL;
    END;
    
    START TRANSACTION;
    
    -- Obtener IDs de usuarios
    SELECT id INTO v_patient_id FROM user WHERE dni = p_patient_dni;
    SELECT id INTO v_created_by_id FROM user WHERE dni = p_created_by_dni;
    
    IF v_patient_id IS NULL OR v_created_by_id IS NULL THEN
        SET p_result_message = 'Error: Usuario no encontrado';
        SET p_invoice_id = 0;
        SET p_invoice_number = NULL;
        ROLLBACK;
    ELSE
        -- Generar número de factura
        SELECT COALESCE(MAX(CAST(SUBSTRING(invoice_number, -8) AS UNSIGNED)), 0) + 1 
        INTO v_next_number 
        FROM invoice 
        WHERE invoice_type = p_invoice_type;
        
        SET p_invoice_number = CONCAT(
            IF(p_invoice_type = 'boleta', 'B001-', 'F001-'),
            LPAD(v_next_number, 8, '0')
        );
        
        -- Calcular totales de servicios
        SET v_service_count = JSON_LENGTH(p_services_json);
        SET v_i = 0;
        
        WHILE v_i < v_service_count DO
            SET v_service_id = JSON_UNQUOTE(JSON_EXTRACT(p_services_json, CONCAT('$[', v_i, '].service_id')));
            SET v_quantity = JSON_UNQUOTE(JSON_EXTRACT(p_services_json, CONCAT('$[', v_i, '].quantity')));
            
            SELECT price INTO v_service_price FROM service WHERE id = v_service_id;
            SET v_line_total = v_service_price * v_quantity;
            SET v_subtotal = v_subtotal + v_line_total;
            
            SET v_i = v_i + 1;
        END WHILE;
        
        -- Calcular impuestos (IGV 18%)
        SET v_tax_amount = ROUND(v_subtotal * 0.18, 2);
        SET v_total_amount = v_subtotal + v_tax_amount;
        
        -- Crear factura
        INSERT INTO invoice (invoice_number, invoice_type, patient_id, appointment_id, issue_date, 
                           due_date, subtotal, tax_amount, total_amount, payment_status, 
                           created_by, created_at, updated_at)
        VALUES (p_invoice_number, p_invoice_type, v_patient_id, p_appointment_id, CURDATE(),
                DATE_ADD(CURDATE(), INTERVAL 30 DAY), v_subtotal, v_tax_amount, v_total_amount,
                'pending', v_created_by_id, NOW(), NOW());
        
        SET p_invoice_id = LAST_INSERT_ID();
        
        -- Insertar detalles de factura
        SET v_i = 0;
        WHILE v_i < v_service_count DO
            SET v_service_id = JSON_UNQUOTE(JSON_EXTRACT(p_services_json, CONCAT('$[', v_i, '].service_id')));
            SET v_quantity = JSON_UNQUOTE(JSON_EXTRACT(p_services_json, CONCAT('$[', v_i, '].quantity')));
            
            SELECT price INTO v_service_price FROM service WHERE id = v_service_id;
            SET v_line_total = v_service_price * v_quantity;
            
            INSERT INTO invoice_detail (invoice_id, service_id, quantity, unit_price, total_price, created_at, updated_at)
            VALUES (p_invoice_id, v_service_id, v_quantity, v_service_price, v_line_total, NOW(), NOW());
            
            SET v_i = v_i + 1;
        END WHILE;
        
        SET p_result_message = 'Factura generada exitosamente';
        COMMIT;
    END IF;
END//

DELIMITER ; = 1 THEN
        RETURN FALSE;
    END IF;
    
    RETURN TRUE;
END//

DELIMITER ;
```

### 5.3 Función para Generar Número de Cita

```sql
DELIMITER //

-- Función para generar número único de cita
CREATE FUNCTION fn_generate_appointment_number(appointment_date DATE)
RETURNS VARCHAR(20)
READS SQL DATA
DETERMINISTIC
BEGIN
    DECLARE next_number INT;
    DECLARE appointment_number VARCHAR(20);
    
    -- Obtener el siguiente número para la fecha
    SELECT COALESCE(MAX(
        CAST(SUBSTRING_INDEX(SUBSTRING_INDEX(notes, 'APT-', -1), '-', 1) AS UNSIGNED)
    ), 0) + 1
    INTO next_number
    FROM appointment
    WHERE appointment_date = appointment_date
    AND notes LIKE '%APT-%';
    
    -- Generar número con formato APT-YYYYMMDD-NNNN
    SET appointment_number = CONCAT(
        'APT-',
        DATE_FORMAT(appointment_date, '%Y%m%d'),
        '-',
        LPAD(next_number, 4, '0')
    );
    
    RETURN appointment_number;
END//

DELIMITER ;
```

### 5.4 Función para Calcular Próxima Cita Disponible

```sql
DELIMITER //

-- Función para encontrar el próximo slot disponible para un odontólogo
CREATE FUNCTION fn_next_available_slot(
    dentist_id BIGINT,
    preferred_date DATE,
    duration_minutes INT
)
RETURNS DATETIME
READS SQL DATA
DETERMINISTIC
BEGIN
    DECLARE next_slot DATETIME DEFAULT NULL;
    DECLARE check_date DATE;
    DECLARE check_time TIME;
    DECLARE end_time TIME;
    DECLARE slot_available BOOLEAN DEFAULT FALSE;
    DECLARE max_iterations INT DEFAULT 30; -- Buscar hasta 30 días
    DECLARE current_iteration INT DEFAULT 0;
    
    SET check_date = preferred_date;
    
    -- Buscar slot disponible
    WHILE NOT slot_available AND current_iteration < max_iterations DO
        -- Obtener horario de trabajo para el día
        SELECT start_time, end_time 
        INTO check_time, end_time
        FROM working_hour
        WHERE working_hour.dentist_id = dentist_id
            AND day_of_week = DAYOFWEEK(check_date) - 1
            AND is_active = TRUE
        LIMIT 1;
        
        -- Si hay horario de trabajo para este día
        IF check_time IS NOT NULL THEN
            -- Buscar slot libre en intervalos de 30 minutos
            WHILE check_time < end_time AND NOT slot_available DO
                -- Verificar si el slot está disponible
                IF NOT EXISTS (
                    SELECT 1 FROM appointment
                    WHERE appointment.dentist_id = dentist_id
                        AND appointment_date = check_date
                        AND appointment_time = check_time
                        AND status_id NOT IN (SELECT id FROM appointment_status WHERE name IN ('cancelled', 'no_show'))
                ) THEN
                    SET next_slot = CONCAT(check_date, ' ', check_time);
                    SET slot_available = TRUE;
                ELSE
                    SET check_time = ADDTIME(check_time, '00:30:00');
                END IF;
            END WHILE;
        END IF;
        
        -- Siguiente día
        SET check_date = DATE_ADD(check_date, INTERVAL 1 DAY);
        SET current_iteration = current_iteration + 1;
        SET check_time = NULL;
    END WHILE;
    
    RETURN next_slot;
END//

DELIMITER ;
```

## 6. Vistas del Sistema

### 6.1 Vista de Usuario Completa

```sql
-- Vista que combina información de usuario con perfiles específicos
CREATE VIEW vw_user_complete AS
SELECT 
    u.id,
    u.dni,
    u.email,
    u.first_name,
    u.last_name,
    CONCAT(u.first_name, ' ', u.last_name) as full_name,
    u.phone,
    u.address,
    u.birth_date,
    fn_calculate_age(u.birth_date) as age,
    u.gender,
    u.is_active,
    u.last_login,
    r.name as role_name,
    r.description as role_description,
    -- Información específica de dentista
    dp.license_number,
    dp.specialization,
    dp.consultation_fee,
    -- Información específica de paciente
    pp.emergency_contact_name,
    pp.emergency_contact_phone,
    pp.blood_type,
    pp.allergies,
    pp.medical_conditions,
    pp.preferred_notification_method
FROM user u
INNER JOIN role r ON u.role_id = r.id
LEFT JOIN dentist_profile dp ON u.id = dp.user_id
LEFT JOIN patient_profile pp ON u.id = pp.user_id;
```

### 6.2 Vista de Agenda Diaria

```sql
-- Vista de agenda diaria con información completa
CREATE VIEW vw_daily_schedule AS
SELECT 
    a.id as appointment_id,
    a.appointment_date,
    a.appointment_time,
    a.duration_minutes,
    TIME_FORMAT(a.appointment_time, '%H:%i') as time_formatted,
    DATE_FORMAT(a.appointment_date, '%d/%m/%Y') as date_formatted,
    -- Información del paciente
    p.dni as patient_dni,
    CONCAT(p.first_name, ' ', p.last_name) as patient_name,
    p.phone as patient_phone,
    fn_calculate_age(p.birth_date) as patient_age,
    -- Información del odontólogo
    d.dni as dentist_dni,
    CONCAT(d.first_name, ' ', d.last_name) as dentist_name,
    dp.specialization as dentist_specialization,
    -- Información de la cita
    a.reason,
    a.notes,
    ast.name as status_name,
    ast.description as status_description,
    ast.color as status_color,
    a.arrived_at,
    a.started_at,
    a.finished_at,
    -- Información del creador
    CONCAT(c.first_name, ' ', c.last_name) as created_by_name
FROM appointment a
INNER JOIN user p ON a.patient_id = p.id
INNER JOIN user d ON a.dentist_id = d.id
LEFT JOIN dentist_profile dp ON d.id = dp.user_id
INNER JOIN appointment_status ast ON a.status_id = ast.id
INNER JOIN user c ON a.created_by = c.id;
```

### 6.3 Vista de Historial Clínico Completo

```sql
-- Vista completa del historial clínico con archivos adjuntos
CREATE VIEW vw_medical_history AS
SELECT 
    mr.id as record_id,
    mr.visit_date,
    -- Información del paciente
    p.dni as patient_dni,
    CONCAT(p.first_name, ' ', p.last_name) as patient_name,
    fn_calculate_age(p.birth_date) as patient_age,
    -- Información del odontólogo
    d.dni as dentist_dni,
    CONCAT(d.first_name, ' ', d.last_name) as dentist_name,
    dp.specialization,
    -- Información del registro médico
    mr.chief_complaint,
    mr.diagnosis,
    mr.treatment_plan,
    mr.treatment_performed,
    mr.observations,
    mr.next_visit_date,
    -- Conteo de archivos adjuntos
    (SELECT COUNT(*) FROM medical_record_file mrf WHERE mrf.medical_record_id = mr.id) as files_count,
    -- Información de la cita asociada
    a.appointment_date,
    a.appointment_time
FROM medical_record mr
INNER JOIN user p ON mr.patient_id = p.id
INNER JOIN user d ON mr.dentist_id = d.id
LEFT JOIN dentist_profile dp ON d.id = dp.user_id
LEFT JOIN appointment a ON mr.appointment_id = a.id;
```

### 6.4 Vista de Reportes Financieros

```sql
-- Vista para reportes de facturación y pagos
CREATE VIEW vw_financial_report AS
SELECT 
    i.id as invoice_id,
    i.invoice_number,
    i.invoice_type,
    i.issue_date,
    i.due_date,
    DATE_FORMAT(i.issue_date, '%Y-%m') as billing_month,
    DATE_FORMAT(i.issue_date, '%Y') as billing_year,
    -- Información del paciente
    p.dni as patient_dni,
    CONCAT(p.first_name, ' ', p.last_name) as patient_name,
    -- Montos
    i.subtotal,
    i.tax_amount,
    i.total_amount,
    i.payment_status,
    i.payment_method,
    i.payment_date,
    -- Días de atraso
    CASE 
        WHEN i.payment_status = 'pending' AND i.due_date < CURDATE() 
        THEN DATEDIFF(CURDATE(), i.due_date)
        ELSE 0
    END as days_overdue,
    -- Estado SUNAT
    i.sunat_status,
    -- Información del creador
    CONCAT(c.first_name, ' ', c.last_name) as created_by_name,
    -- Servicios facturados (concatenados)
    GROUP_CONCAT(
        CONCAT(s.name, ' (', id_detail.quantity, ')')
        SEPARATOR ', '
    ) as services_list
FROM invoice i
INNER JOIN user p ON i.patient_id = p.id
INNER JOIN user c ON i.created_by = c.id
LEFT JOIN invoice_detail id_detail ON i.id = id_detail.invoice_id
LEFT JOIN service s ON id_detail.service_id = s.id
GROUP BY i.id;
```

## 7. Ejemplos de Uso de Procedimientos y Funciones

### 7.1 Crear Usuario Paciente

```sql
-- Ejemplo de uso del procedimiento para crear paciente
CALL sp_create_patient_user(
    '12345678',                    -- DNI
    'juan.perez@email.com',        -- Email
    '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', -- Password hash
    'Juan Carlos',                 -- Nombres
    'Pérez Mamani',               -- Apellidos
    '987654321',                  -- Teléfono
    'Av. El Sol 123, Puno',       -- Dirección
    '1985-03-15',                 -- Fecha de nacimiento
    'male',                       -- Género
    'María Pérez',                -- Contacto de emergencia
    '987123456',                  -- Teléfono de emergencia
    'O+',                         -- Tipo de sangre
    'Penicilina',                 -- Alergias
    @user_id,                     -- OUT: ID del usuario creado
    @result_message               -- OUT: Mensaje de resultado
);

-- Verificar resultado
SELECT @user_id as user_id, @result_message as message;
```

### 7.2 Agendar Cita

```sql
-- Ejemplo de agendamiento de cita
CALL sp_schedule_appointment(
    '12345678',                   -- DNI del paciente
    '87654321',                   -- DNI del odontólogo
    '2024-07-15',                 -- Fecha de la cita
    '10:00:00',                   -- Hora de la cita
    60,                           -- Duración en minutos
    'Consulta de rutina y limpieza dental', -- Motivo
    '11111111',                   -- DNI del usuario que crea la cita
    @appointment_id,              -- OUT: ID de la cita
    @result_message               -- OUT: Mensaje de resultado
);

-- Verificar resultado
SELECT @appointment_id as appointment_id, @result_message as message;
```

### 7.3 Registrar Consulta Médica

```sql
-- Ejemplo de registro de consulta médica
CALL sp_create_medical_record(
    '12345678',                   -- DNI del paciente
    '87654321',                   -- DNI del odontólogo
    1,                            -- ID de la cita
    'Dolor en muela del juicio',  -- Motivo principal
    'Caries profunda en pieza 38', -- Diagnóstico
    'Endodoncia y corona',        -- Plan de tratamiento
    'Anestesia local y preparación inicial', -- Tratamiento realizado
    'Paciente tolera bien el procedimiento', -- Observaciones
    '2024-07-22',                 -- Próxima visita
    @record_id,                   -- OUT: ID del registro
    @result_message               -- OUT: Mensaje de resultado
);

-- Verificar resultado
SELECT @record_id as record_id, @result_message as message;
```

### 7.4 Generar Factura

```sql
-- Ejemplo de generación de factura
SET @services = JSON_ARRAY(
    JSON_OBJECT('service_id', 1, 'quantity', 1),
    JSON_OBJECT('service_id', 3, 'quantity', 1)
);

CALL sp_generate_invoice(
    '12345678',                   -- DNI del paciente
    1,                            -- ID de la cita
    'boleta',                     -- Tipo de comprobante
    @services,                    -- Servicios en formato JSON
    '11111111',                   -- DNI del usuario que crea la factura
    @invoice_id,                  -- OUT: ID de la factura
    @invoice_number,              -- OUT: Número de factura
    @result_message               -- OUT: Mensaje de resultado
);

-- Verificar resultado
SELECT @invoice_id as invoice_id, @invoice_number as invoice_number, @result_message as message;
```

### 7.5 Uso de Funciones

```sql
-- Validar DNI
SELECT fn_validate_dni('12345678') as is_valid_dni;

-- Calcular edad
SELECT fn_calculate_age('1985-03-15') as age;

-- Encontrar próximo slot disponible
SELECT fn_next_available_slot(
    (SELECT id FROM user WHERE dni = '87654321'), 
    CURDATE(), 
    30
) as next_available_slot;

-- Generar número de cita
SELECT fn_generate_appointment_number(CURDATE()) as appointment_number;
```

## 8. Consultas de Reportes Avanzados

### 8.1 Reporte de Productividad de Odontólogos

```sql
-- Reporte mensual de productividad por odontólogo
SELECT 
    d.dni,
    vuc.full_name as dentist_name,
    dp.specialization,
    COUNT(a.id) as total_appointments,
    COUNT(CASE WHEN a.status_id = (SELECT id FROM appointment_status WHERE name = 'completed') THEN 1 END) as completed_appointments,
    COUNT(CASE WHEN a.status_id = (SELECT id FROM appointment_status WHERE name = 'cancelled') THEN 1 END) as cancelled_appointments,
    COUNT(CASE WHEN a.status_id = (SELECT id FROM appointment_status WHERE name = 'no_show') THEN 1 END) as no_show_appointments,
    ROUND(
        COUNT(CASE WHEN a.status_id = (SELECT id FROM appointment_status WHERE name = 'completed') THEN 1 END) * 100.0 / 
        COUNT(a.id), 2
    ) as completion_rate,
    COALESCE(SUM(i.total_amount), 0) as total_revenue,
    COUNT(DISTINCT mr.id) as medical_records_created
FROM user d
INNER JOIN vw_user_complete vuc ON d.id = vuc.id
INNER JOIN dentist_profile dp ON d.id = dp.user_id
LEFT JOIN appointment a ON d.id = a.dentist_id 
    AND a.appointment_date >= DATE_SUB(CURDATE(), INTERVAL 1 MONTH)
LEFT JOIN invoice i ON a.id = i.appointment_id AND i.payment_status = 'paid'
LEFT JOIN medical_record mr ON d.id = mr.dentist_id 
    AND mr.visit_date >= DATE_SUB(CURDATE(), INTERVAL 1 MONTH)
WHERE d.role_id = (SELECT id FROM role WHERE name = 'dentist')
    AND d.is_active = TRUE
GROUP BY d.id, d.dni, vuc.full_name, dp.specialization
ORDER BY total_revenue DESC;
```

### 8.2 Reporte de Pacientes Frecuentes

```sql
-- Pacientes con más visitas en el último año
SELECT 
    p.dni,
    vuc.full_name as patient_name,
    vuc.phone,
    vuc.age,
    COUNT(DISTINCT a.id) as total_appointments,
    COUNT(DISTINCT mr.id) as total_visits,
    MAX(a.appointment_date) as last_appointment,
    COALESCE(SUM(i.total_amount), 0) as total_spent,
    GROUP_CONCAT(DISTINCT SUBSTRING(mr.diagnosis, 1, 50) SEPARATOR '; ') as recent_diagnoses
FROM user p
INNER JOIN vw_user_complete vuc ON p.id = vuc.id
LEFT JOIN appointment a ON p.id = a.patient_id 
    AND a.appointment_date >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
LEFT JOIN medical_record mr ON p.id = mr.patient_id 
    AND mr.visit_date >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
LEFT JOIN invoice i ON p.id = i.patient_id 
    AND i.issue_date >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
    AND i.payment_status = 'paid'
WHERE p.role_id = (SELECT id FROM role WHERE name = 'patient')
    AND p.is_active = TRUE
GROUP BY p.id, p.dni, vuc.full_name, vuc.phone, vuc.age
HAVING total_visits > 2
ORDER BY total_visits DESC, total_spent DESC
LIMIT 20;
```

### 8.3 Reporte de Ingresos por Servicio

```sql
-- Análisis de ingresos por tipo de servicio
SELECT 
        DECLARE v_next_number INT;
    DECLARE v_service_count INT DEFAULT 0;
    DECLARE v_i INT DEFAULT 0;
    DECLARE v_service_id BIGINT;
    DECLARE v_quantity INT;
    DECLARE v_service_price DECIMAL(8,2);
    DECLARE v_line_total DECIMAL(10,2);
    
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_result_message = 'Error: No se pudo generar la factura';
        SET p_invoice_id = 0;
        SET p_invoice_number = NULL;
    END;
    
    START TRANSACTION;
    
    -- Obtener IDs de usuarios
    SELECT id INTO v_patient_id FROM user WHERE dni = p_patient_dni;
    SELECT id INTO v_created_by_id FROM user WHERE dni = p_created_by_dni;
    
    IF v_patient_id IS NULL OR v_created_by_id IS NULL THEN
        SET p_result_message = 'Error: Usuario no encontrado';
        SET p_invoice_id = 0;
        SET p_invoice_number = NULL;
        ROLLBACK;
    ELSE
        -- Generar número de factura
        SELECT COALESCE(MAX(CAST(SUBSTRING(invoice_number, -8) AS UNSIGNED)), 0) + 1 
        INTO v_next_number 
        FROM invoice 
        WHERE invoice_type = p_invoice_type;
        
        SET p_invoice_number = CONCAT(
            IF(p_invoice_type = 'boleta', 'B001-', 'F001-'),
            LPAD(v_next_number, 8, '0')
        );
        
        -- Calcular totales de servicios
        SET v_service_count = JSON_LENGTH(p_services_json);
        SET v_i = 0;
        
        WHILE v_i < v_service_count DO
            SET v_service_id = JSON_UNQUOTE(JSON_EXTRACT(p_services_json, CONCAT('$[', v_i, '].service_id')));
            SET v_quantity = JSON_UNQUOTE(JSON_EXTRACT(p_services_json, CONCAT('$[', v_i, '].quantity')));
            
            SELECT price INTO v_service_price FROM service WHERE id = v_service_id;
            SET v_line_total = v_service_price * v_quantity;
            SET v_subtotal = v_subtotal + v_line_total;
            
            SET v_i = v_i + 1;
        END WHILE;
        
        -- Calcular impuestos (IGV 18%)
        SET v_tax_amount = ROUND(v_subtotal * 0.18, 2);
        SET v_total_amount = v_subtotal + v_tax_amount;
        
        -- Crear factura
        INSERT INTO invoice (invoice_number, invoice_type, patient_id, appointment_id, issue_date, 
                           due_date, subtotal, tax_amount, total_amount, payment_status, 
                           created_by, created_at, updated_at)
        VALUES (p_invoice_number, p_invoice_type, v_patient_id, p_appointment_id, CURDATE(),
                DATE_ADD(CURDATE(), INTERVAL 30 DAY), v_subtotal, v_tax_amount, v_total_amount,
                'pending', v_created_by_id, NOW(), NOW());
        
        SET p_invoice_id = LAST_INSERT_ID();
        
        -- Insertar detalles de factura
        SET v_i = 0;
        WHILE v_i < v_service_count DO
            SET v_service_id = JSON_UNQUOTE(JSON_EXTRACT(p_services_json, CONCAT('$[', v_i, '].service_id')));
            SET v_quantity = JSON_UNQUOTE(JSON_EXTRACT(p_services_json, CONCAT('$[', v_i, '].quantity')));
            
            SELECT price INTO v_service_price FROM service WHERE id = v_service_id;
            SET v_line_total = v_service_price * v_quantity;
            
            INSERT INTO invoice_detail (invoice_id, service_id, quantity, unit_price, total_price, created_at, updated_at)
            VALUES (p_invoice_id, v_service_id, v_quantity, v_service_price, v_line_total, NOW(), NOW());
            
            SET v_i = v_i + 1;
        END WHILE;
        
        SET p_result_message = 'Factura generada exitosamente';
        COMMIT;
    END IF;
END//

DELIMITER ;
