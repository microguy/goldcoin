# Berkeley DB 18.1 Wallet Migration Plan

## Overview
Migration strategy for upgrading Goldcoin wallets from Berkeley DB 4.8 to 18.1 while ensuring complete asset protection.

## Critical Safety Requirements

### 1. Automatic Backup
- **Before ANY migration attempt**, create timestamped backup
- Backup naming: `wallet.dat.bdb4.backup-{timestamp}`
- Store in same directory as original wallet
- Verify backup integrity before proceeding

### 2. Version Detection
- Reliably detect BDB 4.8 wallets
- Check file header/magic bytes
- Validate database structure
- Distinguish between already-migrated and legacy wallets

### 3. Atomic Migration
- Either fully succeed or fully rollback
- No partial states allowed
- Use temporary files during conversion
- Only replace original after full verification

### 4. User Notification
- Clear communication about what's happening
- Progress indicators during migration
- Success/failure messages
- Instructions for manual recovery if needed

### 5. Testing Requirements
- Test with real legacy wallets
- Test with encrypted wallets
- Test with large wallets (many transactions)
- Test failure scenarios and rollback

## Technical Challenges

### Format Incompatibility
- BDB 4.8 and 18.1 have incompatible file formats
- Cannot directly open 4.8 database with 18.1 library
- Need intermediate conversion process

### Data Preservation Requirements
Must preserve ALL wallet data:
- Private keys (encrypted and unencrypted)
- Public keys and addresses
- Transaction history
- Address labels and metadata
- Account information
- Key pool
- Master keys for HD wallets
- Wallet settings and preferences

### Special Considerations
- **Encrypted Wallets**: Must handle without exposing keys
- **HD Wallets**: Preserve derivation paths and master seeds
- **Watch-only Addresses**: Maintain without private keys
- **Multisig**: Preserve all participant information

## Proposed Implementation Strategy

### Phase 1: Detection
```cpp
// Pseudocode
WalletDBVersion detectWalletVersion(const fs::path& walletPath) {
    // Read file header
    // Check BDB magic bytes and version
    // Return BDB_4_8, BDB_18_1, or UNKNOWN
}
```

### Phase 2: Backup
```cpp
// Pseudocode
bool createWalletBackup(const fs::path& walletPath) {
    // Generate timestamp
    // Copy wallet.dat to wallet.dat.bdb4.backup-{timestamp}
    // Verify backup checksum
    // Log backup location
}
```

### Phase 3: Migration
```cpp
// Pseudocode
bool migrateWallet(const fs::path& oldWallet, const fs::path& newWallet) {
    // Open old wallet with compatibility layer
    // Create new BDB 18.1 database
    // Copy all records with validation
    // Maintain encryption if present
}
```

### Phase 4: Verification
```cpp
// Pseudocode
bool verifyMigration(const fs::path& oldWallet, const fs::path& newWallet) {
    // Count keys in both wallets
    // Verify address generation
    // Check transaction counts
    // Validate metadata preservation
}
```

### Phase 5: Cleanup
```cpp
// Pseudocode
bool finalizeMigration(const fs::path& walletPath) {
    // Only after successful verification
    // Move new wallet to original location
    // Keep backup for safety
    // Update wallet version markers
}
```

## Implementation Timeline

### Week 1 Schedule
- **Day 1-2**: Research and prototype version detection
- **Day 3-4**: Implement backup and migration logic
- **Day 5**: Verification and testing
- **Day 6**: Edge cases and error handling
- **Day 7**: Documentation and final testing

## Risk Mitigation

### Backup Strategy
- Multiple backup copies
- Checksums for integrity
- Clear naming with timestamps
- Never delete original until verified

### Failure Handling
- Automatic rollback on any error
- Detailed error logging
- User instructions for manual recovery
- Support for external backup restoration

### Testing Protocol
1. Create test wallets with BDB 4.8
2. Add various transaction types
3. Test migration process
4. Verify all data preserved
5. Test rollback scenarios

## User Experience

### Automatic Detection and Prompt
```
Goldcoin Core has detected a legacy wallet format (Berkeley DB 4.8).
This needs to be upgraded to the new format (Berkeley DB 18.1) for compatibility.

Your wallet will be automatically backed up before migration.
Backup location: /path/to/wallet.dat.bdb4.backup-20250821-180623

Would you like to proceed with the migration? [Yes/No]
```

### Progress Feedback
```
[===========         ] 55% - Migrating wallet entries...
Migrated: 1,234 of 2,245 records
Time remaining: ~30 seconds
```

### Success Message
```
✓ Wallet successfully migrated to Berkeley DB 18.1
✓ Backup saved at: /path/to/wallet.dat.bdb4.backup-20250821-180623
✓ All 2,245 records verified and intact

Your wallet is now ready to use with Goldcoin Core 0.17.0
```

## Code Integration Points

### Files to Modify
- `src/wallet/db.cpp` - Database version detection
- `src/wallet/walletdb.cpp` - Migration logic
- `src/wallet/wallet.cpp` - Integration point
- `src/init.cpp` - Startup detection and prompt

### New Files to Create
- `src/wallet/walletmigration.cpp` - Core migration logic
- `src/wallet/walletmigration.h` - Migration interfaces
- `src/test/walletmigration_tests.cpp` - Unit tests

## Security Considerations

### Private Key Protection
- Never expose unencrypted keys during migration
- Use secure memory for temporary storage
- Clear sensitive data after use

### File Permissions
- Maintain original wallet file permissions
- Secure backup files (0600 on Linux)
- Prevent unauthorized access during migration

### Audit Trail
- Log all migration attempts
- Record success/failure states
- Maintain migration history

## Success Criteria

1. **Zero Data Loss** - Every wallet element preserved
2. **Automatic Process** - Minimal user interaction
3. **Safe Rollback** - Can always restore original
4. **Performance** - Migration under 5 minutes for typical wallet
5. **Compatibility** - Works with all wallet types

## Notes

- Berkeley DB 18.1.40 selected for long-term support
- Migration is one-way (cannot downgrade after migration)
- Backups should be retained indefinitely
- Consider offering manual migration tool as well

---

*Document created: August 21, 2025*  
*Branch: feature/bdb-18-wallet-migration*  
*Priority: CRITICAL - Protecting user assets is paramount*