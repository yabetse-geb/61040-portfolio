# Exercise 1
1. **Invariants:** The first invariant is that the number in each Request is greater than or equal to 0. The second invariant is that for each item in a registry, the sum of all purchases plus the current request count always equals the total number of units ever requested for that item (including any additions via addItem). This second invariant is more important because it ensures that purchased gifts were gifts that were requested, and this prevents over and under purchasing. The action most affected by this invariant is purchase and it perserves it by having the requirment that the item that is being purchased is active and has a request for the item with at least count, this prevents over purchasing. And the fact, and it perserves it by decrementing the count in the matching request, ensuring number purchases of an item + current count of item= original count for item in registry.

2. **Fixing an action:** The action that can break this invariant is purchase. This can happen if it decrements the request count by the wrong amount or allows more units to be purchased than are available. To fix it, purchase should check that the request has at least the purchased count, create the purchase record, and decrement the request count by exactly that amount.

3. **Inferring behavior:** A registry can be opened and closed repeatedly, because both actions simply flip the active flag while ensuring the registry exists. This allows flexibility for the recipient; for example, a request for winter coats might be open in the summer when prices are lower, but closed during the high-demand season to prevent purchases at a higher cost.

4. **Registry deletion:** This could matter in practice if there is limited space in the system that records registrys, as outdated/uncessary registrys could take too much space.

5. **Queries:** Two likely, commmon quries are "Which items have been purchased, and who purchased them" (for registry ownwer), and "Which items still need to be purchased" (for a gift giver).

6. **Hiding Purchases:** Add a flag showPurchases to the registry. When it is false, the owner cannot see the set of Purchases, and the counts shown for requests are not decreased, so they don’t know what was bought. Purchases still update the real counts and records, just not the counts and records visible to the recipient. So, the owner’s view is filtered to keep the element of surprise.

7. **Generic types:** This is preferable because it allows this type of registry to be more reusable to fit different stores or systems.
