# DATAPOLIS v4.0 - PARTE 6: DUE DILIGENCE CORPORATIVA

## Sistema Integral de Debida Diligencia

**Versión**: 4.0.0 | **Fecha**: Febrero 2026

---

# 1. VISIÓN GENERAL

El Sistema Due Diligence integra **8 módulos especializados** para análisis integral en transacciones M&A, inversiones y relaciones comerciales.

## 1.1 Módulos

| Módulo | Descripción |
|--------|-------------|
| DD01-M&A | Fusiones y Adquisiciones |
| DD02-FIN | Due Diligence Financiera |
| DD03-LEG | Due Diligence Legal |
| DD04-TAX | Due Diligence Tributaria |
| DD05-OPR | Due Diligence Operacional |
| DD06-TEC | Due Diligence Tecnológica |
| DD07-ESG | Due Diligence ESG |
| DD08-REP | Due Diligence Reputacional |

---

# 2. IMPLEMENTACIÓN COMPLETA

```python
# =============================================================================
# DUE DILIGENCE SUITE - COMPLETE IMPLEMENTATION
# =============================================================================

from dataclasses import dataclass, field
from typing import List, Dict, Any, Optional
from datetime import date, datetime
from enum import Enum
from decimal import Decimal
import uuid

# ENUMS
class DealType(str, Enum):
    ACQUISITION = "acquisition"
    MERGER = "merger"
    JOINT_VENTURE = "joint_venture"
    MINORITY_INVESTMENT = "minority_investment"

class DealStage(str, Enum):
    SCREENING = "screening"
    PRELIMINARY_DD = "preliminary_dd"
    CONFIRMATORY_DD = "confirmatory_dd"
    NEGOTIATION = "negotiation"
    CLOSING = "closing"

class FindingSeverity(str, Enum):
    CRITICAL = "critical"
    HIGH = "high"
    MEDIUM = "medium"
    LOW = "low"
    INFO = "informational"

# MODELS
@dataclass
class Deal:
    id: str = field(default_factory=lambda: str(uuid.uuid4()))
    code: str = ""
    name: str = ""
    deal_type: DealType = DealType.ACQUISITION
    stage: DealStage = DealStage.SCREENING
    target_name: str = ""
    enterprise_value: Optional[Decimal] = None
    equity_value: Optional[Decimal] = None

@dataclass
class DDFinding:
    id: str = field(default_factory=lambda: str(uuid.uuid4()))
    deal_id: str = ""
    workstream: str = ""
    title: str = ""
    description: str = ""
    severity: FindingSeverity = FindingSeverity.MEDIUM
    quantified_impact: Optional[Decimal] = None
    status: str = "open"

# SERVICES
class DDScoringEngine:
    SEVERITY_POINTS = {
        'critical': 40, 'high': 25, 'medium': 10, 'low': 3, 'info': 0
    }
    
    def calculate_score(self, findings: List[DDFinding]) -> Dict:
        total_points = sum(
            self.SEVERITY_POINTS.get(f.severity.value, 10) 
            for f in findings
        )
        score = max(0, 100 - (total_points / 2))
        
        if score >= 80: classification = "Low Risk"
        elif score >= 60: classification = "Medium Risk"
        elif score >= 40: classification = "High Risk"
        else: classification = "Critical Risk"
        
        return {
            'score': round(score, 1),
            'classification': classification,
            'total_findings': len(findings),
            'critical_count': sum(1 for f in findings if f.severity == FindingSeverity.CRITICAL)
        }

class QualityOfEarningsAnalyzer:
    def calculate_adjusted_ebitda(
        self,
        reported_ebitda: Decimal,
        adjustments: List[Dict]
    ) -> Dict:
        total_adj = sum(Decimal(str(a['amount'])) for a in adjustments)
        adjusted = reported_ebitda + total_adj
        return {
            'reported': float(reported_ebitda),
            'adjustments': adjustments,
            'total_adjustment': float(total_adj),
            'adjusted_ebitda': float(adjusted),
            'adjustment_percentage': float(total_adj / reported_ebitda * 100) if reported_ebitda else 0
        }

class WorkingCapitalAnalyzer:
    def calculate_nwc(self, data: Dict) -> Dict:
        ca = data.get('ar', 0) + data.get('inventory', 0) + data.get('prepaid', 0)
        cl = data.get('ap', 0) + data.get('accrued', 0) + data.get('deferred', 0)
        nwc = ca - cl
        return {
            'current_assets': ca,
            'current_liabilities': cl,
            'net_working_capital': nwc
        }

class ValuationService:
    def calculate_dcf(
        self,
        fcf: List[float],
        wacc: float,
        terminal_growth: float,
        net_debt: float
    ) -> Dict:
        wacc_d = wacc / 100
        g = terminal_growth / 100
        
        pv_fcf = sum(f / ((1 + wacc_d) ** (i+1)) for i, f in enumerate(fcf))
        terminal_fcf = fcf[-1] * (1 + g)
        tv = terminal_fcf / (wacc_d - g)
        pv_tv = tv / ((1 + wacc_d) ** len(fcf))
        
        ev = pv_fcf + pv_tv
        equity = ev - net_debt
        
        return {
            'enterprise_value': round(ev),
            'equity_value': round(equity),
            'pv_fcf': round(pv_fcf),
            'pv_terminal': round(pv_tv),
            'terminal_percentage': round(pv_tv / ev * 100, 1)
        }
```

# 3. DATABASE SCHEMA

```sql
CREATE SCHEMA IF NOT EXISTS due_diligence;

CREATE TABLE due_diligence.deals (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code VARCHAR(30) UNIQUE NOT NULL,
    name VARCHAR(200) NOT NULL,
    deal_type VARCHAR(30) NOT NULL,
    stage VARCHAR(30) DEFAULT 'screening',
    target_name VARCHAR(200),
    enterprise_value DECIMAL(18,2),
    equity_value DECIMAL(18,2),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE due_diligence.findings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    deal_id UUID REFERENCES due_diligence.deals(id),
    workstream VARCHAR(30) NOT NULL,
    title VARCHAR(300) NOT NULL,
    description TEXT,
    severity VARCHAR(20) NOT NULL,
    quantified_impact DECIMAL(18,2),
    status VARCHAR(20) DEFAULT 'open',
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE due_diligence.checklists (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    deal_id UUID REFERENCES due_diligence.deals(id),
    workstream VARCHAR(30) NOT NULL,
    total_items INTEGER DEFAULT 0,
    completed_items INTEGER DEFAULT 0,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE due_diligence.dataroom_documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    deal_id UUID REFERENCES due_diligence.deals(id),
    folder_path VARCHAR(500),
    filename VARCHAR(300) NOT NULL,
    file_size_bytes BIGINT,
    view_count INTEGER DEFAULT 0,
    uploaded_at TIMESTAMPTZ DEFAULT NOW()
);
```

# 4. API ENDPOINTS

```python
from fastapi import APIRouter, HTTPException
from typing import List, Optional

router = APIRouter(prefix="/api/v1/dd", tags=["Due Diligence"])

@router.post("/deals")
async def create_deal(data: dict):
    """Crear nuevo deal."""
    pass

@router.get("/deals/{deal_id}")
async def get_deal(deal_id: str):
    """Obtener deal."""
    pass

@router.get("/deals/{deal_id}/findings")
async def list_findings(deal_id: str, severity: Optional[str] = None):
    """Listar hallazgos."""
    pass

@router.post("/deals/{deal_id}/findings")
async def add_finding(deal_id: str, finding: dict):
    """Agregar hallazgo."""
    pass

@router.get("/deals/{deal_id}/scoring")
async def get_scoring(deal_id: str):
    """Obtener scoring consolidado."""
    pass

@router.get("/deals/{deal_id}/valuation")
async def get_valuation(deal_id: str):
    """Obtener valoración."""
    pass

@router.get("/deals/{deal_id}/qoe")
async def get_quality_of_earnings(deal_id: str):
    """Obtener Quality of Earnings."""
    pass
```

---

**FIN PARTE 6**
