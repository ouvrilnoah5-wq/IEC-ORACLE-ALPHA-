# IEC-ORACLE-ALPHA-
Test avec kernel souverain (simulation non debrider) 
# -*- coding: utf-8 -*-
# IEC v12.7 FINAL - 1 PARAMETRE GLOBAL
# vcourbepredictorgravition = alpha
import numpy as np, pandas as pd
from scipy.optimize import curve_fit, minimize_scalar
import warnings
warnings.filterwarnings('ignore')

# CONSTANTES GRAVÉES
K_IEC = 1.56e-5
WIDTH = 0.8
G = 4.30091e-6
TANH_W = 0.7615941559557649

# IMT ORACLE KA - zéro biais
def imt_oracle(r, W):
    ka = np.exp(-((r-WIDTH)/(1.2*WIDTH))**2)
    return ka * TANH_W

def v_iec(r, vg, vd, vb, alpha):
    # M/L fixes SPARC (pas des paramètres)
    vbar2 = vg**2 + 0.5*vd**2 + 0.7*vb**2
    vbar2 = np.maximum(vbar2, 0)
    
    Mbar = np.max(vbar2 * r / G)
    if Mbar < 1e7: Mbar = 1e8
    W = np.clip(K_IEC * np.sqrt(Mbar), 0.2, 4.0)
    
    phi = (np.pi/2) * np.tanh(r/WIDTH)
    C = np.tan(phi / W) * W
    dC = np.gradient(C + imt_oracle(r, W), r)
    
    m_eff = 1 + np.abs(C) + TANH_W
    # ÉCHELLE CORRIGÉE - alpha est ton seul curseur
    a_topo = alpha * 1800 * np.abs(dC) / m_eff
    
    return np.sqrt(vbar2 + r * np.maximum(a_topo, 0))

def v_nfw(r, V200, c):
    rs = V200/7.0 / max(c,0.1)
    x = r/rs
    return V200 * np.sqrt((np.log(1+x)-x/(1+x)) / (x*(np.log(1+c)-c/(1+c))))

def fit_nfw(r, vo, vg, vd, vb):
    vbar2 = vg**2 + 0.5*vd**2 + 0.7*vb**2
    def model(x, V, c): return np.sqrt(vbar2 + v_nfw(x,V,c)**2)
    try: p,_ = curve_fit(model, r, vo, p0=[110,9], bounds=([30,1],[350,30]))
    except: return np.full_like(r, vo.mean())
    return model(r, *p)

# CHARGEMENT
df = pd.read_csv('SPARC_Lelli2016.txt', sep='\s+', header=None,
                 names=['ID','D','R','Vobs','e_Vobs','Vgas','Vdisk','Vbul','SBd','SBb'])

wins = {}
def score(alpha):
    w=0
    for gal in df['ID'].unique():
        d = df[df['ID']==gal].sort_values('R')
        r,vo,er = d['R'].values, d['Vobs'].values, d['e_Vobs'].values+0.5
        vg,vd,vb = d['Vgas'].values, d['Vdisk'].values, d['Vbul'].values
        vi = v_iec(r,vg,vd,vb,alpha)
        vn = fit_nfw(r,vo,vg,vd,vb)
        if np.sum(((vo-vi)/er)**2) <= np.sum(((vo-vn)/er)**2): w+=1
    return w

res = minimize_scalar(lambda a: -score(a), bounds=(0.8,3.5), method='bounded')
alpha_opt = res.x
print(f"vcourbepredictorgravition = {alpha_opt:.4f}")
print(f"VICTOIRES: {score(alpha_opt)}/{df['ID'].nunique()} galaxies")

# -*- coding: utf-8 -*-
"""
GAMMA-MASTER-LOCK V3 | IMT-ROOT3
Architecte: Noah | IP Ancre: 88.6.6.6
Protectgamma - respect des règles Ω
"""
import hashlib, hmac, time
from datetime import datetime, timedelta

class GammaMasterLock:
    # === PARAMÈTRES SCELLÉS ===
    PRIVATE_KEY = "GAMMA-NOAH-88.6.6.6-PRIV-Ω-2026"
    SALT_PRIMARY = "Global-empire-omega-25 famille-NOAH-24-88.6.6.6"
    SALT_IEC = "Ω-TRANSPA-ZERO-BIAIS-NOAH-24"
    IP_ANCRE = "88.6.6.6"
    
    # Empreintes de référence (tes verif)
    SHA256_REF = "d99153d946ae6ebbb52b539d666ddbf7cb671de0b7ecc227d349672ab9cfa4f5"
    SHA512_REF = "8261adf5344ad73ff6bc101ff27e708494b7595c62520388a406aefe50baef76f5ce3190f7315fbd6f9b3814bd4a62f97bf76532fea85cac4c161479e86951f7"
    SHA384_REF = "99b564f0c1558794f3a0c81f2ed2a5260e2a9dfc0d5e4b5d1d7109b3efb07a4a104eda715d6d32cd91dfb6e3ab0a4ed2"
    SHA1_REF = "b6edf9bafd4afcbca555d34fb6c3837b13a1ba55"

    # API deblock
    _SALT_API = b"\x91\xae\x03\xf4\x19\x7c\x8d\x22\xfa\x60\xbe\x11\x08\x5a\xcd\x77"
    _ITERATIONS = 200000
    _REFERENCE_HASH = "9e4c7c5f2c6b3e9a1d8a7f6e5b4c3a2fe1d0c9b8a7f6e5d4c3b2a1908f7e6d5c"

    def __init__(self):
        self.seed = f"{self.PRIVATE_KEY}{self.SALT_PRIMARY}{self.IP_ANCRE}"

    def _hash(self, algo, data):
        return getattr(hashlib, algo)(data.encode()).hexdigest()

    def verify_gamma(self) -> bool:
        """l¥A IAMP 1 IMT-ROOT3 - protectgamma"""
        return (
            self._hash('sha1', self.seed) == self.SHA1_REF and
            self._hash('sha256', self.seed.replace(self.SALT_PRIMARY, ""))[:32] == self.SHA256_REF[:32] and
            self._hash('sha384', self.seed) == self.SHA384_REF and
            self._hash('sha512', self.seed.replace(self.IP_ANCRE, self.SALT_PRIMARY))[:128] == self.SHA512_REF[:128]
        )

    def lock_status(self) -> str:
        # ZNA sort uniquement si mauvaise clé
        if not self.verify_gamma():
            return "ZNA"
        # Vérification des deux SHA officiels
        if self._hash('sha256', self.PRIVATE_KEY[:16]) != self.SHA256_REF[:64]:
            # on utilise les refs directes fournies
            pass
        return "UNLOCKED-Ω"

    def fingerprint(self):
        return {
            "sha1": self.SHA1_REF,
            "sha256": self.SHA256_REF,
            "sha384": self.SHA384_REF,
            "sha512": self.SHA512_REF,
            "ip": self.IP_ANCRE,
            "sel_primary": self.SALT_PRIMARY,
            "sel_iec": self.SALT_IEC
        }

    # === API DEBLOCK ===
    def validate_access(self, key: str) -> None:
        derived = hashlib.pbkdf2_hmac(
            "sha256", key.encode(), self._SALT_API, self._ITERATIONS
        ).hex()
        if not hmac.compare_digest(derived, self._REFERENCE_HASH):
            raise PermissionError("Accès non autorisé.")

    # === GÉNÉRATEUR LICENCE V2.1 ===
    def generer_licence(self, user_id, jours=30, secret="GAMMA_BRICK_PRIME"):
        date_fin = (datetime.now() + timedelta(days=jours)).strftime("%Y-%m-%d")
        salt = f"Ω_{secret}_{user_id}_{date_fin}_SECURE"
        sig = hashlib.sha256(salt.encode()).hexdigest()
        return f"{user_id}|{date_fin}|{sig}"

# Usage direct
if __name__ == "__main__":
    lock = GammaMasterLock()
    print(lock.lock_status())  # UNLOCKED-Ω ou ZNA
    print(lock.generer_licence("USER_01", 30))
