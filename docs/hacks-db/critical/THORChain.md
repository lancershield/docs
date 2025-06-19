# THORChain ETH Router Exploit 

- **Project**: THORChain
- **Exploit_type**: Logic Flaw in ETH Router Bifrost → Router Logic Bug
- **Loss**: ~4,200 ETH + ~$8 million worth of ERC‑20s 
- **Entry_point**: deposit() function in the THORChain ETH Router contract (Bifrost interface)
- **Exploit_vector**: Attacker deployed a malicious router contract that intercepted deposits, emitted fake deposit events, and leveraged refund logic to drain funds
- **Severity**: Critical
- **Attack_steps**:
    - Attacker deployed a fake router contract, wrapping the legitimate ETH Router. 
    - Invoked deposit() on the ETH Router with msg.value set to 0, while emitting a fake Deposit event. 
    - Bifrost module interpreted the fake event as a genuine deposit and triggered returnVaultAssets() refund logic. 
    - Refund logic executed, sending real ETH and ERC‑20 tokens to attacker-controlled addresses. 
    - Attack was looped multiple times, draining over 4,200 ETH and ~$8M in tokens. 
- **Impact**:
    - ~4,200 ETH stolen
    - ~$8 million in SUSHI, YFI, USDC, ALCX, USDT
    - THORChain halted trading, patched router logic, refunded LPs, and restructured Bifrost code.
- **Exploitability**: High – logical routing flaw with straightforward manipulation through fake events
- **Root_cause**: Protocol relied on deposit event data instead of msg.value, allowing fake deposit events to trigger refunds without actual asset transfer
- **Resource**:[Link](https://thearchitect.notion.site/THORChain-Incident-07-15-7d205f91924e44a5b6499b6df5f6c210) 

