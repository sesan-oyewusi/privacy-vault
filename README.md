# PrivacyVault Protocol

A decentralized privacy-preserving pool system built on Stacks blockchain that enables anonymous transactions through collective fund mixing.

## Overview

PrivacyVault empowers users to maintain financial privacy by creating and participating in collaborative funding pools. Users can deposit funds, form privacy groups, and collectively redistribute assets to break transaction trails. The protocol implements robust security measures including daily transaction limits, automated participant verification, and emergency circuit breakers.

### Key Features

- **Multi-tier Pool Management**: Create and join privacy pools with up to 10 participants
- **Daily Transaction Limits**: Built-in rate limiting for enhanced security
- **Dynamic Fee Structure**: 2% protocol fee on pool distributions
- **Owner-controlled Revenue**: Protocol fee collection and emergency controls
- **Regulatory Compliance**: Transparent pool mechanics and verifiable fund distribution

## Architecture

### Core Components

- **Privacy Pools**: Collaborative funding pools that mix participant funds
- **Balance Management**: Individual user balance tracking with withdrawal capabilities
- **Rate Limiting**: Daily transaction limits to prevent abuse
- **Fee Collection**: Protocol revenue generation through mixing fees
- **Emergency Controls**: Owner-operated circuit breakers for security

### Constants & Limits

| Parameter | Value | Description |
|-----------|--------|-------------|
| `MAX_DAILY_LIMIT` | 10,000 STX | Maximum daily transaction limit per user |
| `MAX_POOL_PARTICIPANTS` | 10 | Maximum participants per privacy pool |
| `MAX_TRANSACTION_AMOUNT` | 1,000,000 STX | Maximum single transaction amount |
| `MIN_POOL_AMOUNT` | 0.1 STX | Minimum contribution to join a pool |
| `MIXING_FEE_PERCENTAGE` | 2% | Protocol fee on pool distributions |

## Smart Contract Functions

### Public Functions

#### `initialize()`

Initializes the PrivacyVault protocol. Must be called by contract owner before any other operations.

#### `deposit(amount: uint)`

Securely deposits STX tokens into user's protocol balance with rate limiting.

**Parameters:**

- `amount`: Amount in microSTX to deposit

**Requirements:**

- Amount must be > 0 and <= MAX_TRANSACTION_AMOUNT
- Must not exceed daily transaction limit
- Contract must be initialized and not paused

#### `withdraw(amount: uint)`

Withdraws STX tokens from user's protocol balance to their wallet.

**Parameters:**

- `amount`: Amount in microSTX to withdraw

**Requirements:**

- Sufficient balance in protocol
- Must not exceed daily transaction limit
- Contract must be initialized and not paused

#### `create-mixer-pool(pool-id: uint, initial-amount: uint)`

Creates a new privacy pool with initial funding.

**Parameters:**

- `pool-id`: Unique identifier for the pool (< 1000)
- `initial-amount`: Initial pool contribution (>= MIN_POOL_AMOUNT)

**Requirements:**

- Pool ID must not already exist
- Sufficient balance for initial contribution
- Amount must meet minimum pool requirements

#### `join-mixer-pool(pool-id: uint, amount: uint)`

Joins an existing privacy pool with a contribution.

**Parameters:**

- `pool-id`: ID of the pool to join
- `amount`: Contribution amount (>= MIN_POOL_AMOUNT)

**Requirements:**

- Pool must exist and be active
- Pool must not be full (< 10 participants)
- User must not already be a participant
- Sufficient balance for contribution

#### `distribute-pool-funds(pool-id: uint)`

Executes the privacy pool distribution, redistributing funds among participants.

**Parameters:**

- `pool-id`: ID of the pool to distribute

**Process:**

1. Calculates 2% protocol fee from total pool amount
2. Distributes remaining funds equally among participants
3. Deactivates the pool after distribution
4. Accumulates protocol fees

#### `toggle-contract-pause()`

Emergency function to pause/unpause the protocol (Owner only).

#### `withdraw-protocol-fees()`

Withdraws accumulated protocol fees to contract owner (Owner only).

### Read-Only Functions

#### `get-user-balance(user: principal) -> uint`

Returns the current protocol balance for a user.

#### `get-daily-limit-remaining(user: principal) -> uint`

Calculates remaining daily transaction limit for a user.

#### `get-contract-status() -> { is-paused: bool, is-initialized: bool, total-protocol-fees: uint }`

Returns current protocol status information.

#### `get-pool-details(pool-id: uint) -> { ... }`

Retrieves complete information about a specific privacy pool.

## Error Codes

| Code | Constant | Description |
|------|----------|-------------|
| 1000 | ERR-NOT-AUTHORIZED | Unauthorized access attempt |
| 1001 | ERR-INVALID-AMOUNT | Invalid transaction amount |
| 1002 | ERR-INSUFFICIENT-BALANCE | Insufficient user balance |
| 1003 | ERR-CONTRACT-NOT-INITIALIZED | Contract not initialized |
| 1004 | ERR-ALREADY-INITIALIZED | Contract already initialized |
| 1005 | ERR-POOL-FULL | Privacy pool at maximum capacity |
| 1006 | ERR-DAILY-LIMIT-EXCEEDED | Daily transaction limit exceeded |
| 1007 | ERR-INVALID-POOL | Pool does not exist or is invalid |
| 1008 | ERR-DUPLICATE-PARTICIPANT | User already participating in pool |
| 1009 | ERR-INSUFFICIENT-POOL-FUNDS | Insufficient funds in pool |
| 1010 | ERR-POOL-NOT-READY | Pool not ready for distribution |

## Development Setup

### Prerequisites

- [Clarinet](https://docs.hiro.so/stacks/clarinet) - Smart contract development tool
- [Node.js](https://nodejs.org/) (v18+) - For testing framework
- [Git](https://git-scm.com/) - Version control

### Installation

1. Clone the repository:

```bash
git clone https://github.com/sesan-oyewusi/privacy-vault.git
cd privacy-vault
```

2. Install dependencies:

```bash
npm install
```

3. Verify setup:

```bash
clarinet check
```

### Testing

Run the test suite:

```bash
npm test
```

Run tests with coverage and cost analysis:

```bash
npm run test:report
```

Watch mode for development:

```bash
npm run test:watch
```

### Contract Validation

Check contract syntax and structure:

```bash
clarinet check
```

Format contract code:

```bash
clarinet fmt --in-place
```

## Usage Examples

### Basic Workflow

1. **Initialize Protocol** (Owner only):

```clarity
(contract-call? .privacy-vault initialize)
```

2. **Deposit Funds**:

```clarity
(contract-call? .privacy-vault deposit u1000000) ;; 1 STX
```

3. **Create Privacy Pool**:

```clarity
(contract-call? .privacy-vault create-mixer-pool u1 u500000) ;; 0.5 STX
```

4. **Join Existing Pool**:

```clarity
(contract-call? .privacy-vault join-mixer-pool u1 u500000) ;; 0.5 STX
```

5. **Distribute Pool Funds**:

```clarity
(contract-call? .privacy-vault distribute-pool-funds u1)
```

6. **Withdraw Funds**:

```clarity
(contract-call? .privacy-vault withdraw u800000) ;; 0.8 STX after fees
```

### Query Functions

Check your balance:

```clarity
(contract-call? .privacy-vault get-user-balance tx-sender)
```

View pool details:

```clarity
(contract-call? .privacy-vault get-pool-details u1)
```

Check daily limit remaining:

```clarity
(contract-call? .privacy-vault get-daily-limit-remaining tx-sender)
```

## Security Considerations

### Rate Limiting

- Daily transaction limits prevent abuse and flash loan attacks
- Per-user tracking ensures fair usage across participants

### Pool Security

- Maximum participant limits prevent pool bloat
- Duplicate participation prevention ensures fair distribution
- Pool state validation prevents invalid operations

### Access Control

- Owner-only functions for protocol management
- Emergency pause functionality for crisis response
- Transparent fee collection and withdrawal

### Fund Safety

- Contract balance tracking prevents double-spending
- Atomic operations ensure consistency
- Withdrawal validation prevents unauthorized access

## Roadmap

- [ ] Multi-token support beyond STX
- [ ] Advanced pool mixing algorithms
- [ ] Governance token integration
- [ ] Cross-chain privacy bridging
- [ ] Enhanced anonymity features
- [ ] Automated pool management

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature-name`
3. Commit changes: `git commit -am 'Add feature'`
4. Push to branch: `git push origin feature-name`
5. Submit a pull request

### Development Guidelines

- Follow Clarity best practices
- Add comprehensive tests for new features
- Update documentation for API changes
- Ensure all tests pass before submitting PR

## License

This project is licensed under the ISC License - see the LICENSE file for details.
