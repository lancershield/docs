# Hegic Uninitialized Variable Bug 

- **Project**: Hegic
- **Exploit_type**: Uninitialized State Variable
- **Loss**: ~$280,000 worth of locked ETH 
- **Entry_point**: Smart contract constructor with incorrect constructor() declaration
- **Exploit_vector**: Funds sent to contract became permanently inaccessible due to constructor bug
- **Severity**: Medium
- **Attack_steps**:
    - Hegic deployed a smart contract with function constructor() instead of the correct constructor() syntax.
    - As a result, the constructor logic was never executed during deployment.
    - Critical state variables such as the contract owner were never initialized.
    - Users began depositing ETH into the contract, expecting normal functionality.
    - Without a valid owner or callable withdraw function, the ETH became irretrievable.
    - ~$280K worth of ETH was effectively locked in the contract permanently.
- **Impact**: No funds stolen, but $280K of ETH became permanently inaccessible
- **Exploitability**: Low (accidental design flaw, not an attack)
- **Root_cause**: Incorrect use of constructor syntax in Solidity ≥0.4.22 caused initialization failure
- **Resource**:[Link](https://decrypt.co/35038/hegics-molly-wintermute-im-paying-a-high-price-for-the-mainnet-first-approach-to-building)