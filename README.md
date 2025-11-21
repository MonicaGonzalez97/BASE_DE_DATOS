CREATE DATABASE IF NOT EXISTS DB_CertificadorDeporteOfi
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_ai_ci;

USE DB_CertificadorDeporteOfi;

-- 1. ClienteAPI 
CREATE TABLE ClienteAPI (
    IdCliente BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    NombreCliente VARCHAR(150) NOT NULL,
    NIT VARCHAR(20) NOT NULL,
    ApiKeyHash VARCHAR(255) NOT NULL,
    Estado ENUM('ACTIVO','INACTIVO') NOT NULL DEFAULT 'ACTIVO',
    CreadoEn TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    ActualizadoEn TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (IdCliente),
    UNIQUE KEY uq_cliente_nit (NIT)
) ENGINE=InnoDB;

-- 2. Serie 
CREATE TABLE Serie (
    IdSerie BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    IdCliente BIGINT UNSIGNED NOT NULL,
    Codigo VARCHAR(20) NOT NULL,
    SiguienteCorrelativo INT UNSIGNED NOT NULL DEFAULT 1,
    Activa BOOLEAN NOT NULL DEFAULT TRUE,
    CreadoEn TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    ActualizadoEn TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (IdSerie),
    UNIQUE KEY uq_serie_cliente_codigo (IdCliente, Codigo),
    CONSTRAINT fk_serie_cliente FOREIGN KEY (IdCliente) REFERENCES ClienteAPI(IdCliente)
        ON UPDATE CASCADE ON DELETE RESTRICT
) ENGINE=InnoDB;

-- 3. FacturaCertificada 
CREATE TABLE FacturaCertificada (
    IdFacturaCertificada BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    IdCliente BIGINT UNSIGNED NOT NULL,
    FacturaExternaId VARCHAR(64) NOT NULL,
    IdempotenciaKey VARCHAR(64) NOT NULL,
    DataOriginal JSON NOT NULL,
    Estado ENUM('PENDIENTE','CERTIFICADA','RECHAZADA','ANULADA') NOT NULL DEFAULT 'PENDIENTE',
    ErrorCodigo VARCHAR(64) NULL,
    ErrorMensaje VARCHAR(400) NULL,
    NumeroAutorizacion CHAR(36) NULL,
    IdSerie BIGINT UNSIGNED NULL,
    Serie VARCHAR(20) NULL,
    Correlativo INT UNSIGNED NULL,
    FechaCertificacion DATETIME NULL,
    MontoTotal DECIMAL(12,2) NULL,
    AnulacionMotivo VARCHAR(250) NULL,
    FechaAnulacion DATETIME NULL,
    CreadoEn TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    ActualizadoEn TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (IdFacturaCertificada),
    UNIQUE KEY uq_idem (IdCliente, IdempotenciaKey),
    UNIQUE KEY uq_autorizacion (NumeroAutorizacion),
    UNIQUE KEY uq_serie_correlativo (IdSerie, Correlativo),
    KEY ix_estado (Estado, CreadoEn),
    KEY ix_factura_externa (FacturaExternaId),
    CONSTRAINT fk_factcert_cliente FOREIGN KEY (IdCliente) REFERENCES ClienteAPI(IdCliente)
        ON UPDATE CASCADE ON DELETE RESTRICT,
    CONSTRAINT fk_factcert_serie FOREIGN KEY (IdSerie) REFERENCES Serie(IdSerie)
        ON UPDATE CASCADE ON DELETE RESTRICT,
    CHECK (JSON_VALID(DataOriginal))
) ENGINE=InnoDB;

-- 4. BitacoraCertificacion
CREATE TABLE BitacoraCertificacion (
    IdBitacora BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    IdFacturaCertificada BIGINT UNSIGNED NOT NULL,
    Accion ENUM('RECIBIDA','VALIDACION_OK','VALIDACION_ERROR','CORRELATIVO_ASIGNADO','CERTIFICADA','ANULADA') NOT NULL,
    Fecha DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    Detalle TEXT NULL,
    Datos JSON NULL,
    PRIMARY KEY (IdBitacora),
    CONSTRAINT fk_bitacora_fact FOREIGN KEY (IdFacturaCertificada) REFERENCES FacturaCertificada(IdFacturaCertificada)
        ON UPDATE CASCADE ON DELETE CASCADE,
    CHECK (Datos IS NULL OR JSON_VALID(Datos))
) ENGINE=InnoDB;

-- 5. CertificacionAPI 
CREATE TABLE EmisorCertificacionAPI (
    IdEmisorCertificacion INT NOT NULL AUTO_INCREMENT,
    IdFacturaCertificada BIGINT UNSIGNED NOT NULL,
    NumeroAutorizacion CHAR(36) NOT NULL,
    Serie VARCHAR(20) NOT NULL,
    Correlativo INT UNSIGNED NOT NULL,
    FechaCertificacion DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (IdEmisorCertificacion),
    UNIQUE KEY uq_numero_autorizacion (NumeroAutorizacion),
    UNIQUE KEY uq_serie_correlativo (Serie, Correlativo),
    CONSTRAINT fk_emisorcert_factura FOREIGN KEY (IdFacturaCertificada) REFERENCES FacturaCertificada(IdFacturaCertificada)
        ON UPDATE CASCADE ON DELETE RESTRICT
) ENGINE=InnoDB;



SHOW TABLES;

SELECT * FROM FACTURACERTIFICADA;

ALTER TABLE emisorcertificacionapi
RENAME TO CertificacionesAprobadas;
