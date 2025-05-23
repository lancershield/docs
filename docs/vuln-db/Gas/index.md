 # 🟠 Gas Severity 

- Refers to smart contract patterns that cause excessive gas consumption, which may degrade performance, block functionality, or increase user costs.

- Includes unbounded loops, inefficient storage writes, redundant state changes, and gas-heavy fallbacks.

- Not always exploitable directly, but can lead to DoS (Denial-of-Service) under high activity or prevent critical functions (e.g., withdraw, executeProposal) from completing.

- May also impact L2s and rollups more severely due to stricter gas constraints, making optimization essential.

- Severity varies:

- Low when inefficiencies are minor and cosmetic

- Medium–High when they can break key flows or create griefing opportunities

- Fixes are strongly recommended in user-facing, time-sensitive, or mass-iteration functions to protect usability and scalability.

- Gas severity is not about security leaks — it’s about execution risk, denial of service, and long-term operational efficiency.

