
## Arquitectura Empresarial Exhaustiva - Vertical de Cumplimiento Normativo

**Versión**: 4.0.0  
**Fecha**: Febrero 2026  
**Clasificación**: Documentación Técnica Empresarial  
**Autor**: DATAPOLIS SpA - Arquitectura de Soluciones

---

# ÍNDICE PARTE 5

1. [Visión General del Compliance Suite](#1-visión-general)
2. [Módulo C01-GRC: Governance, Risk & Compliance](#2-módulo-c01-grc)
3. [Módulo C02-AML: Anti-Money Laundering](#3-módulo-c02-aml)
4. [Módulo C03-KYC: Know Your Customer](#4-módulo-c03-kyc)
5. [Módulo C04-PEP: Politically Exposed Persons](#5-módulo-c04-pep)
6. [Módulo C05-SAR: Suspicious Activity Reporting](#6-módulo-c05-sar)
7. [Módulo C06-CTR: Currency Transaction Reporting](#7-módulo-c06-ctr)
8. [Módulo C07-FATCA: Foreign Account Tax Compliance](#8-módulo-c07-fatca)
9. [Módulo C08-CRS: Common Reporting Standard](#9-módulo-c08-crs)
10. [Módulo C09-GDPR: Data Protection & Privacy](#10-módulo-c09-gdpr)
11. [Arquitectura Técnica Integrada](#11-arquitectura-técnica)
12. [Implementación de Código Completa](#12-implementación-código)

---

# 1. VISIÓN GENERAL

## 1.1 Propósito del Compliance Suite

El Compliance Suite de DATAPOLIS integra **9 módulos especializados** para gestión integral del cumplimiento normativo en instituciones financieras, inmobiliarias y corporativas en Chile y Latinoamérica.

### 1.1.1 Regulaciones Cubiertas

| Jurisdicción | Regulación | Módulo(s) |
|--------------|------------|-----------|
| Chile | Ley 19.913 (UAF) | C02-AML, C05-SAR |
| Chile | Ley 20.393 (Resp. Penal PJ) | C01-GRC |
| Chile | Circular UAF N°57 | C03-KYC, C04-PEP |
| Chile | NCG 380 CMF | C01-GRC |
| Internacional | FATF 40 Recommendations | C02-AML |
| USA | FATCA | C07-FATCA |
| OCDE | CRS | C08-CRS |
| UE | GDPR | C09-GDPR |
| Chile | Ley 19.628 (Datos) | C09-GDPR |

### 1.1.2 Arquitectura del Suite

```
┌─────────────────────────────────────────────────────────────────────┐
│                     COMPLIANCE SUITE v4.0                            │
├─────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │
│  │  C01-GRC    │  │  C02-AML    │  │  C03-KYC    │                  │
│  │ Governance  │  │Anti-Lavado  │  │   Clientes  │                  │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘                  │
│         │                │                │                          │
│  ┌──────┴────────────────┴────────────────┴──────┐                  │
│  │           MOTOR DE REGLAS COMPLIANCE           │                  │
│  │      (Drools + Python Rules Engine)            │                  │
│  └──────┬────────────────┬────────────────┬──────┘                  │
│         │                │                │                          │
│  ┌──────┴──────┐  ┌──────┴──────┐  ┌──────┴──────┐                  │
│  │  C04-PEP    │  │  C05-SAR    │  │  C06-CTR    │                  │
│  │  Personas   │  │  Reportes   │  │Transacciones│                  │
│  │  Expuestas  │  │ Sospechosos │  │  Efectivo   │                  │
│  └─────────────┘  └─────────────┘  └─────────────┘                  │
│                                                                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │
│  │  C07-FATCA  │  │  C08-CRS    │  │  C09-GDPR   │                  │
│  │  US Fiscal  │  │ Intercambio │  │  Privacidad │                  │
│  │  Compliance │  │ Automático  │  │    Datos    │                  │
│  └─────────────┘  └─────────────┘  └─────────────┘                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

# 2. MÓDULO C01-GRC: GOVERNANCE, RISK & COMPLIANCE

## 2.1 Descripción Funcional

### 2.1.1 Propósito
Sistema integrado de gestión de gobierno corporativo, riesgos y cumplimiento basado en el marco COSO ERM 2017 y ISO 31000:2018.

### 2.1.2 Funcionalidades Principales

| ID | Funcionalidad | Descripción |
|----|---------------|-------------|
| GRC-01 | Gestión de Políticas | Repositorio versionado de políticas con workflow de aprobación |
| GRC-02 | Matriz de Riesgos | Identificación, evaluación y tratamiento de riesgos |
| GRC-03 | Controles Internos | Mapeo de controles a riesgos con testing periódico |
| GRC-04 | Incidentes | Registro, investigación y remediación de incidentes |
| GRC-05 | Auditorías | Planificación y ejecución de auditorías internas |
| GRC-06 | Obligaciones | Calendario de obligaciones regulatorias |
| GRC-07 | KRIs/KPIs | Dashboard de indicadores de riesgo y desempeño |
| GRC-08 | Reportería | Generación automática de informes al directorio |

### 2.1.3 Metodología de Evaluación de Riesgos

```
Riesgo Inherente = Probabilidad × Impacto

Probabilidad (P):
├── 5: Casi Cierto (>90%)
├── 4: Probable (70-90%)
├── 3: Posible (30-70%)
├── 2: Improbable (10-30%)
└── 1: Raro (<10%)

Impacto (I):
├── 5: Catastrófico (>$10M, pérdida licencia)
├── 4: Mayor ($1M-$10M, sanción CMF)
├── 3: Moderado ($100K-$1M, multa)
├── 2: Menor ($10K-$100K, observación)
└── 1: Insignificante (<$10K, hallazgo)

Riesgo Residual = Riesgo Inherente × (1 - Efectividad Control)

Efectividad Control:
├── 95%: Excelente (automatizado, probado)
├── 80%: Bueno (documentado, consistente)
├── 60%: Adecuado (existe, variable)
├── 40%: Débil (informal, inconsistente)
└── 20%: Inexistente (sin control)
```

## 2.2 Arquitectura Técnica C01-GRC

### 2.2.1 Modelo de Datos

```sql
-- =============================================================================
-- COMPLIANCE SUITE: C01-GRC - GOVERNANCE, RISK & COMPLIANCE
-- PostgreSQL 16 Schema
-- =============================================================================

-- Schema dedicado
CREATE SCHEMA IF NOT EXISTS compliance_grc;
SET search_path TO compliance_grc, public;

-- -----------------------------------------------------------------------------
-- CATÁLOGOS BASE
-- -----------------------------------------------------------------------------

CREATE TABLE risk_categories (
    id SERIAL PRIMARY KEY,
    code VARCHAR(20) UNIQUE NOT NULL,