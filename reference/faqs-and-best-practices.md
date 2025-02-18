---
hidden: true
icon: comment-question
---

# FAQs & best practices

## Integration best practices

1. **Data Validation**
   * Validate all parameters before submission
   * Check token decimals and amounts
   * Verify addresses are on correct chain
2. **Error Handling**
   * Implement proper try-catch blocks
   * Handle RPC errors and timeouts
   * Verify transaction receipts
3. **Gas Optimization**
   * Batch queries when possible
   * Use multicall for read operations
   * Monitor gas prices for transactions
4. **Security**
   * Never expose private keys
   * Validate module states before interaction
   * Check permissions and access control
5. **Monitoring**
   * Track all relevant events
   * Monitor asset state changes
   * Keep audit logs of operations
