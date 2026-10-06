%%writefile main.py
import os
import hashlib
from fastapi import FastAPI, UploadFile, File, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel

# تهيئة تطبيق BlockCert واجهة Swagger التفاعلية
app = FastAPI(
    title="🔗 BlockCert System",
    description="نظام توثيق الشهادات الأكاديمية وحماية المستقلين عبر البلوكشين والذكاء الاصطناعي",
    version="1.0.0",
    docs_url="/docs"
)

# السماح بالاتصال من الهواتف والمتصفحات
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# محاكاة سجل البلوكشين ومحافظ الضمان المالي
BLOCKCHAIN_LEDGER = {}
ESCROW_WALLETS = {}

class EscrowContract(BaseModel):
    contract_id: str
    freelancer_wallet: str
    client_wallet: str
    amount: float
    status: str = "HOLD"

# محاكاة ذكاء وربط نموذج Gemini لقرأة وفحص الشهادات والأختام
async def analyze_document_with_gemini(file_bytes: bytes, lang: str) -> dict:
    return {
        "status": "VALID",
        "institution": "جامعة BlockCert التقنية",
        "degree": "Master of Computer Science",
        "anti_fraud_score": 0.99,
        "message": "تم فحص الأختام برمجياً: المستند سليم وموثق 100%"
    }

@app.get("/", tags=["العامة"])
def read_root():
    return {"message": "Welcome to BlockCert API", "status": "Running"}

@app.post("/api/v1/certificates/verify", tags=["1. توثيق الشهادات الأكاديمية"])
async def verify_and_anchor_certificate(lang: str = "ar", file: UploadFile = File(...)):
    try:
        file_content = await file.read()
        ai_analysis = await analyze_document_with_gemini(file_content, lang)
        cert_hash = hashlib.sha256(file_content).hexdigest()
        
        if cert_hash in BLOCKCHAIN_LEDGER:
            return {
                "verified": True,
                "source": "Blockchain Ledger",
                "message": "تنبيه: هذه الشهادة مسجلة مسبقاً في البلوكشين وغير معدلة",
                "details": BLOCKCHAIN_LEDGER[cert_hash]
            }
            
        record = {
            "hash": cert_hash,
            "metadata": ai_analysis,
            "timestamp": "2026-10-06T11:00:00Z"
        }
        BLOCKCHAIN_LEDGER[cert_hash] = record
        
        return {
            "verified": True,
            "source": "AI & Blockchain Initial Anchoring",
            "message": "نجاح: تم فحص الشهادة بالذكاء الاصطناعي وتوثيق بصمتها في البلوكشين بنجاح",
            "blockchain_receipt": record
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.post("/api/v1/escrow/create", tags=["2. محفظة الضمان للمستقلين"])
def create_escrow_contract(contract: EscrowContract):
    if contract.contract_id in ESCROW_WALLETS:
        raise HTTPException(status_code=400, detail="Contract ID already exists.")
    ESCROW_WALLETS[contract.contract_id] = contract.dict()
    return {"status": "SUCCESS", "message": "تم حجز أموال المشروع قانونياً في محفظة الضمان (Fiat-to-Smart Contract)"}

@app.post("/api/v1/escrow/release/{contract_id}", tags=["2. محفظة الضمان للمستقلين"])
def release_funds(contract_id: str):
    if contract_id not in ESCROW_WALLETS:
        raise HTTPException(status_code=404, detail="Contract not found.")
    contract = ESCROW_WALLETS[contract_id]
    contract["status"] = "RELEASED"
    return {"status": "SUCCESS", "message": "أرسل العقد الذكي إشارة الصرف: تم تحويل الأموال لحساب المستقل البنكي فوراً"}
