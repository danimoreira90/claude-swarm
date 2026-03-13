---
name: refactor-clean
description: >
  Safe refactoring skill. Extract, rename, restructure without breaking functionality.
  Follows the de-sloppify pattern: separate cleanup pass after implementation.
  Always: test-covered before refactoring, tests still pass after.
---

# Refactor Clean Skill

> Refactoring is changing how code is written without changing what it does. Tests prove you didn't break it.

## The Golden Rule

**Never refactor without a test suite.** Before any significant refactor:
1. Write tests that cover the current behavior
2. Make sure they pass
3. Refactor
4. Ensure tests still pass

## The De-Sloppify Pattern (From ECC)

After implementing a feature, run a cleanup pass:

```bash
# Step 1: Implementation (builder agent)
claude -p "Implement JWT refresh token rotation per PLAN.md."

# Step 2: De-sloppify pass (refactor-clean skill)
claude -p "Review files changed in the previous commit.
Remove:
- Unnecessary defensive checks (trusting framework guarantees)
- Over-engineered abstractions for one-time use
- console.log/print debug statements
- TODO comments without ticket references
- Dead code (unreachable, commented-out code)
Keep:
- Real business logic
- Security-critical checks
- Comments explaining WHY (not WHAT)
Run tests after cleanup to verify nothing broke."
```

## Refactoring Patterns

### Extract Function

```typescript
// Before: long function doing multiple things
async function processOrder(orderId: string, userId: string) {
  const order = await db.orders.findUnique({ where: { id: orderId } });
  if (!order) throw new NotFoundException();
  if (order.userId !== userId) throw new ForbiddenException();
  if (order.status !== 'pending') throw new BadRequestException('Order already processed');

  const payment = await stripe.paymentIntents.create({
    amount: order.totalCents,
    currency: 'usd',
    metadata: { orderId },
  });

  await db.orders.update({
    where: { id: orderId },
    data: { status: 'processing', stripePaymentIntentId: payment.id },
  });

  await emailService.send(userId, 'order-processing', { orderId });
  return { paymentIntentId: payment.id };
}

// After: extracted, named responsibilities
async function processOrder(orderId: string, userId: string) {
  const order = await this.validateOrderForProcessing(orderId, userId);
  const payment = await this.createStripePaymentIntent(order);
  await this.updateOrderToProcessing(orderId, payment.id);
  await this.sendProcessingEmail(userId, orderId);
  return { paymentIntentId: payment.id };
}
```

### Replace Conditional with Polymorphism

```typescript
// Before: type-based switch
function processNotification(notification: Notification) {
  switch (notification.type) {
    case 'email': return sendEmail(notification);
    case 'sms': return sendSMS(notification);
    case 'push': return sendPush(notification);
  }
}

// After: polymorphism
interface NotificationHandler {
  send(notification: Notification): Promise<void>;
}

class EmailHandler implements NotificationHandler { ... }
class SMSHandler implements NotificationHandler { ... }
class PushHandler implements NotificationHandler { ... }
```

### Flatten Nesting

```typescript
// Before: deeply nested
async function validateUser(id: string) {
  const user = await findUser(id);
  if (user) {
    if (user.isActive) {
      if (user.emailVerified) {
        return user;
      } else {
        throw new Error('Email not verified');
      }
    } else {
      throw new Error('User inactive');
    }
  } else {
    throw new NotFoundException();
  }
}

// After: early returns (guard clauses)
async function validateUser(id: string) {
  const user = await findUser(id);
  if (!user) throw new NotFoundException();
  if (!user.isActive) throw new Error('User inactive');
  if (!user.emailVerified) throw new Error('Email not verified');
  return user;
}
```

## Safe Refactoring Checklist

```
Before:
[ ] Tests exist for the code being refactored
[ ] Tests pass before refactoring starts
[ ] Clear goal: what exactly is being improved?
[ ] Scope is limited (refactor one thing at a time)

During:
[ ] Small incremental changes (not big-bang rewrites)
[ ] Run tests after each change
[ ] Don't add functionality while refactoring
[ ] Don't change tests while refactoring (exception: renaming)

After:
[ ] All tests still pass
[ ] Coverage didn't drop
[ ] TypeScript errors: none
[ ] Lint errors: none
[ ] Behavior is identical (black-box same)
```

## What NOT to Refactor in a Feature PR

Refactoring unrelated code in a feature PR:
- Makes PRs harder to review
- Increases risk of breaking unrelated functionality
- Mixes "what changed" signals in git blame

**Rule**: Create a separate PR for refactoring. Keep feature PRs focused.
