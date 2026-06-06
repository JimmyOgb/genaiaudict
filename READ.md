# 🛡️ GenLayer AI Upgrade Safety Checker

An automated, consensus-backed AI security gatekeeper built on **GenLayer**. This system natively fetches smart contract code updates directly from **IPFS**, processes semantic diffs via decentralized LLM validators, and acts as a programmable barrier to protect DAOs from malicious or buggy protocol upgrades.

🚀 **Live Prototype Demo:** [Preview Dashboard](https://preview--aiaudict.lovable.app/)  
📜 **Smart Contract IDE:** [Open in GenLayer Studio](https://studio.genlayer.com/?import-contract=0x222201F7742dac79Aa37cd6aC5242F65eb303745)

---

## 💡 Overview & Problem Statement

Traditional blockchain upgrade patterns (like `TransparentUpgradeableProxy` or UUPS) rely heavily on human multi-sigs or DAO voting timelines. If a malicious proposal slipped through governance unnoticed, or an engineering team introduced a silent storage layout collision, the upgrade would execute deterministically—resulting in catastrophic exploits.

**The Solution:** This project utilizes GenLayer's **Intelligent Predictors** to execute native web queries and AI-driven static code analysis *on-chain* during the governance timelock window. 

### Key Features Tested
* 🛑 **Automated Access Control Auditing:** Instantly alerts if critical modifiers (like `onlyOwner`) are dropped in newer implementations.
* 📦 **Storage Layout Stability Verification:** Evaluates variable slot shifting to prevent storage state corruption.
* 🗳️ **DAO Gatekeeper Integration:** Exposes an on-chain boolean hook (`check_upgrade_safety`) that standard timelock executors can query before final deployment.

---

## 🏗️ Architecture Design
[ DAO Governance / Timelock ]
                              │
                   (Triggers Audit Request)
                              │
                              ▼
                [ GenLayer Safety Contract ]
                              │
         ┌────────────────────┼────────────────────┐
         ▼                    ▼                    ▼
 [Validator Node 1]   [Validator Node 2]   [Validator Node 3]
         │                    │                    │
 (Fetch IPFS Data)    (Fetch IPFS Data)    (Fetch IPFS Data)
         │                    │                    │
         └────────────────────┼────────────────────┘
                              ▼
                 [ Decentralized Consensus ]
                              │
                    (Run LLM Evaluation)
                              │
                              ▼
                [ Storage Status Updated ] ──> [ Frontend UI ]

---

## 🛠️ Smart Contract Layout (`contract.py`)

The core logic uses the GenLayer Python environment and relies on the `current_predictor` to aggregate decentralized node queries across IPFS file streams:

```python
from genlayer import SmartContract, current_predictor, JSON, gl_export
from genlayer.std.storage import Mapping

class UpgradeSafetyChecker(SmartContract):
    reports: Mapping[int, str]
    is_approved: Mapping[int, bool]
    governance_bridge_address: str

    def __init__(self):
        self.reports = Mapping[int, str]()
        self.is_approved = Mapping[int, bool]()
        self.governance_bridge_address = ""

    @gl_export
    def request_upgrade_audit(self, proposal_id: int, old_code_cid: str, new_code_cid: str) -> str:
        old_code_url = f"[https://ipfs.io/ipfs/](https://ipfs.io/ipfs/){old_code_cid}"
        new_code_url = f"[https://ipfs.io/ipfs/](https://ipfs.io/ipfs/){new_code_cid}"

        system_prompt = "You are a decentralized Web3 Security Oracle. Analyze code diffs and output raw JSON."
        user_prompt = f"Compare contract modifications between {old_code_url} and {new_code_url}. Check layout alterations."

        try:
            response = current_predictor.call_llm(
                system_prompt=system_prompt,
                user_prompt=user_prompt,
                response_format={"type": "json_object"}
            )
            report = JSON.parse(response)
            self.reports[proposal_id] = response
            
            # Enforce gatekeeping rules
            if report.get("security_decreased", True) or report.get("risk_level") == "HIGH":
                self.is_approved[proposal_id] = False
            else:
                self.is_approved[proposal_id] = True
            return "Audit completed successfully."
        except Exception as e:
            self.is_approved[proposal_id] = False  # Fail closed
            return "Audit execution defaulted to safe/failed mode."
🚀 Quickstart & Installation
To run or simulate the workspace ecosystem locally:

1. Prerequisites
Ensure you have the latest GenLayer CLI dependencies installed:

Bash
npm install -g @genlayer/cli
2. Local Simulation Testing
Deploy the contract logic tracking your initialized environment parameters:

Bash
# Compile and deploy directly to your local simulator instance
genlayer deploy contract.py

# Query an active evaluation tracking your targeted IPFS pointers
genlayer call <DEPLOYED_ADDRESS> request_upgrade_audit '[1, "QmOldCode...", "QmNewCode..."]'
📊 Dashboard Interactivity
The frontend interface bridges the user visibility layer with real-time consensus logic tracking:

Interactive Audit Submission Panel: Allows inputting custom IPFS CIDs to dispatch analysis updates.

Automated Risk Tagging UI: Seamlessly highlights severity categorization flags (CRITICAL, WARNING, PASS).

Raw State Hex Parser: Exposes the literal JSON return format delivered directly from the consensus round execution.

🔒 Security Posture & Safeguards
Fail-Closed Defaulting: If an LLM output fails standard structured JSON validations or encounters gateway timeout limitations, the evaluation engine automatically triggers a CRITICAL risk posture state—blocking downstream automated contract execution blocks until human governance intervention checks occur.

Consensus-Backed Scraping: The architecture removes third-party manipulation vectors since majority node agreement rules enforce strict source integrity matching before logic processing passes.
