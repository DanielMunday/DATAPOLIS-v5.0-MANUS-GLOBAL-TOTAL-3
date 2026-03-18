## PARTE 2: VERTICAL FINTECH (FINANCE)

---

# 3. VERTICAL FINTECH (FINANCE)

## 3.1 Visión General

La vertical FinTech de DATAPOLIS aborda las necesidades de instituciones financieras en cuatro dominios críticos: Open Finance (NCG 514), Riesgo de Crédito (Basel IV), Valoración de Colaterales, y Scoring Crediticio. Esta vertical está diseñada para bancos, cooperativas de ahorro, fintechs, y administradoras de fondos.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        VERTICAL FINTECH                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    MÓDULOS FINTECH                              │   │
│  ├─────────────────────────────────────────────────────────────────┤   │
│  │                                                                 │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐│   │
│  │  │   M01-OF    │  │   M02-CS    │  │   M03-RWA   │  │ M04-COL ││   │
│  │  │   Open      │  │   Credit    │  │   Risk      │  │Collateral│   │
│  │  │  Finance    │  │  Scoring    │  │  Weighted   │  │ Valuation│   │
│  │  │             │  │  Basel IV   │  │   Assets    │  │          ││   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘│   │
│  │                                                                 │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │   │
│  │  │   M05-PF    │  │   M06-LIQ   │  │       M07-STRESS        │ │   │
│  │  │  Portfolio  │  │  Liquidity  │  │     Stress Testing      │ │   │
│  │  │ Management  │  │    Risk     │  │     & Scenarios         │ │   │
│  │  └─────────────┘  └─────────────┘  └─────────────────────────┘ │   │
│  │                                                                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  INTEGRACIONES:                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ CMF → NCG 514  │  SBIF → Basel  │  SII → Impuestos  │  TGSS    │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3.2 Módulo M01-OF: Open Finance (NCG 514)

### 3.2.1 Descripción

El módulo Open Finance implementa el marco regulatorio de la NCG 514 de la CMF (Comisión para el Mercado Financiero), habilitando el Sistema de Finanzas Abiertas de Chile. Permite la portabilidad segura de datos financieros entre instituciones bajo consentimiento explícito del cliente.

### 3.2.2 Funcionalidades Detalladas

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     M01-OF: OPEN FINANCE                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  FUNCIONALIDAD 1: GESTIÓN DE CONSENTIMIENTO                            │
│  ════════════════════════════════════════════                          │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │ • Captura de consentimiento granular (por tipo de dato)           │ │
│  │ • Almacenamiento criptográfico de evidencia de consentimiento     │ │
│  │ • Revocación en tiempo real con propagación a terceros            │ │
│  │ • Dashboard de consentimientos activos por cliente                │ │
│  │ • Historial auditable de cambios de consentimiento                │ │
│  │ • Expiración automática configurable (30/60/90/365 días)          │ │
│  │ • Notificaciones pre-expiración al cliente                        │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  FUNCIONALIDAD 2: APIS DE DATOS FINANCIEROS                            │
│  ═══════════════════════════════════════════                           │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │ ENDPOINTS IMPLEMENTADOS:                                          │ │
│  │                                                                   │ │
│  │ /accounts                                                         │ │
│  │ ├── GET  /accounts                    → Lista cuentas del cliente │ │
│  │ ├── GET  /accounts/{id}               → Detalle de cuenta         │ │
│  │ └── GET  /accounts/{id}/transactions  → Movimientos               │ │
│  │                                                                   │ │
│  │ /credit                                                           │ │
│  │ ├── GET  /credit/cards                → Tarjetas de crédito       │ │
│  │ ├── GET  /credit/loans                → Préstamos activos         │ │
│  │ └── GET  /credit/history              → Historial crediticio      │ │
│  │                                                                   │ │
│  │ /investments                                                      │ │
│  │ ├── GET  /investments/portfolios      → Portafolios               │ │
│  │ ├── GET  /investments/positions       → Posiciones actuales       │ │
│  │ └── GET  /investments/transactions    → Operaciones               │ │
│  │                                                                   │ │
│  │ /insurance                                                        │ │
│  │ ├── GET  /insurance/policies          → Pólizas vigentes          │ │
│  │ └── GET  /insurance/claims            → Siniestros                │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  FUNCIONALIDAD 3: SEGURIDAD FAPI 2.0                                   │
│  ═══════════════════════════════════                                   │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │ IMPLEMENTACIÓN DE SEGURIDAD:                                      │ │
│  │                                                                   │ │