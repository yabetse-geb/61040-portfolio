# Exercise 1
1. **Invariants:** The first invariant is that the number in each Request is greater than or equal to 0. The second invariant is that for each item in a registry, the sum of all purchases plus the current request count always equals the total number of units ever requested for that item (including any additions via addItem). This second invariant is more important because it ensures that purchased gifts were gifts that were requested, and this prevents over and under purchasing. The action most affected by this invariant is purchase and it perserves it by having the requirment that the item that is being purchased is active and has a request for the item with at least count, this prevents over purchasing. And the fact, and it perserves it by decrementing the count in the matching request, ensuring number purchases of an item + current count of item= original count for item in registry.

2. **Fixing an action:** The action that can break this invariant is purchase. This can happen if it decrements the request count by the wrong amount or allows more units to be purchased than are available. To fix it, purchase should check that the request has at least the purchased count, create the purchase record, and decrement the request count by exactly that amount.

3. **Inferring behavior:** A registry can be opened and closed repeatedly, because both actions simply flip the active flag while ensuring the registry exists. This allows flexibility for the recipient; for example, a request for winter coats might be open in the summer when prices are lower, but closed during the high-demand season to prevent purchases at a higher cost.

4. **Registry deletion:** This could matter in practice if there is limited space in the system that records registrys, as outdated/uncessary registrys could take too much space.

5. **Queries:** Two likely, commmon quries are "Which items have been purchased, and who purchased them?" (for registry ownwer), and "Which items still need to be purchased?" (for a gift giver).

6. **Hiding Purchases:** Add a flag showPurchases to the registry. When it is false, the owner cannot see the set of Purchases, and the counts shown for requests are not decreased, so they don’t know what was bought. Purchases still update the real counts and records, just not the counts and records visible to the recipient. So, the owner’s view is filtered to keep the element of surprise.

7. **Generic types:** This is preferable because it allows this type of registry (the GiftRegistration concept) to be more reusable and to fit different stores or systems.


# Exercise 2
1. state:
a set of Users with
    -a username String
    -a password String
2. register (username: String, password: String): User
    requires:
        -there exists no user in Users, with the same username
    effects:
        creates a new User with username and password

 authenticate (username: String, password: String): User
    requires:
        - there exist a User in Users with username and password passed in

3. There are no users in Users set that have the same username, this is preserved by the requires specification for the register action

4. Updated concept below:
  - **concept:** PasswordAccessToken[User, Token]
  - **purpose:** limit access to known users

  - **principle:** A user registers with a username and password, creating a pending account and a secret token. After receiving the token (e.g., via email), the user provides it to confirm their account. Once confirmed, they can authenticate with that same username and password and be treated each time as the same user.
  - **state:**
    - a set of Users with:
        - a username String
        - a password String
        - an isConfirmed Flag
        - a scope:String
        - a confirmationToken: Token

  - **actions:**
    - register (username: String, password: String): User
        - requires: there exists no user in Users, with the same username
        - effects: creates a new User with username and password

    - confirm (username:String, token:Token)
        - requies: a user exists with this username whose isConfirmed flag is false
        - effect: set the isConfirmed flag for this user to true






# Exercise 3
  - **concept:** PasswordAccessToken[User]
  - **purpose:** enable users to authenticate using generated tokens that grant scoped, time-limited access to resources
  - **principle:** after a user generates a personal access token, they can present its secret string to authenticate as that user, while the token has not expired and its scope permits the requested action. After the token expires, it is of no use.
  - **state:**
    - a set of AccessTokens with:
        - an owner:User
        - an ExperationDate:String
        - a name:String
        - a scope:String
        - tokenString: String

  - **actions:**
    - generateAccessToken(user: User, name: String, scope: String, expirationDate: Date): AccessToken

        - requires: the user exists, no AccessToken with name=name and owner=user, and expirationDate is in the future

        - effect: create a new AccessToken with a fresh, globally unique tokenString, owner = user, and the provided name, scope, expirationDate; add it to the set; return the created AccessToken

    - deleteAccessToken(user: User, name: String)

        - requires: the user exists and has an AccessToken with that name

        - effect: removes the AccessToken from the set of AccessTokens

    - authenticate(tokenString: String): (user: User)

        - requires: there exists an AccessToken with the given tokenString; its expirationDate has not passed.and its scope allows authentication

        - effect: returns the owner user associated with that Accesstoken

    - update(user:User, name:String, newScope: String, newExperiation:Date)
        - requires: **user** exists and has an **AccessToken** with **name**=name
        - effect: modify that AccessTokens **scope** and **experationDate** to newScope and **newExperiationDate**


Password access tokens are different from regular password authentication in a few important ways. Instead of giving full access like a password does, each token is tied to a user but can have its own scope that limits what it can do. Tokens can also expire or be deleted without changing the user’s password. Each token has a name you use to manage it and a secret string that actually authenticates you, unlike a password which does both at once. The GitHub docs could be clearer about this distinction, highlight scope and expiration, and maybe include a small diagram showing tokens, users, and permissions.


# Exercise 4
## **Billable Hours Tracking**
  - **concept:** BillableHoursReporting[User, Project]
  - **purpose:** To enable accurate and automated tracking of time spent by users on specific projects for billing and record-keeping.
  - **principle:** An employee starts a timer for a specific project and provides a work description. When the task is finished, the employee stops the timer, which records a completed work session. If an employee forgets to stop a session, they or a manager can later manually edit the entry to ensure its accuracy.
  - **state:**
    - a set of Sessions with:
        - an employee:User
        - a project:Project
        - an startTime:Time
        - an endTime:Time (optional)
        - a description:String

  - **actions:**
    - startSession(user: User, project:Project, workDescription: String): Session

        - requires: There is no existing Session for user that has a startTime but no endTime.

        - effect: Creates a new Session with the given user, project, description, and the current time as its startTime. The endTime is not set. Returns the newly created session.

    - endSession(session: Session)

        - requires: the given session exists

        - effect: Updates the given session by setting its endTime to the current time.

    - editSession(session:Session, newStartTime:Time, newEndTime:Time, newDescription:String)

        - requires: the given session exists

        - effect: updates the session's start and end time, and description to the given values.

    - deleteSession(session:Session)

        - requires: the given session exists

        - effect: removes session from the set of Sessions


## **Conference Room Booking**
  - **concept:** ConferenceRoomBooking[User]
  - **purpose:** Make it easy for users to find and book available conference rooms at specific times, avoid scheduling conflicts, and ensure shared spaces are used efficiently.
  - **principle:** An administrator first populates the system with the available conference rooms and their properties (e.g., capacity). When a user needs a room, they can view a room's schedule or search for an available room at a desired time. The user then creates a booking for an available time slot, which makes that slot unavailable to others. If their plans change, the user can cancel the booking, freeing the room for others to use.
  - **state:**
    - a set of Rooms with:
        - a name: String
    - a set of Bookings with:
        - a booker:User
        - a room: Room
        - a title: String
        - a startTime: Time
        - an endTime: Time

  - **actions:**
    - addRoom(name: String): Session

        - requires: There doesn't already exist a Room with the passed in name

        - effect: Creates a new Room with the given details and adds it to the set of Rooms. This is typically an administrative action.

    - createBooking(user: User, room: Room, title: String, startTime: Time, endTime: Time): Booking

        - requires: if the room exists in the set of Bookings it doesn't have an overlapping start and end time as the startTime and endTime passed in

        - effect: creates a new Booking with the passed in values, updates the set of Bookings with the new Booking, and returns this created Booking

    - cancelBooking(booking:Booking)

        - requires: the given booking exists in the set of Bookings

        - effect: removes booking from the set of Bookings

    - removeRoom(room:Room)

        - requires: room has no current or future Bookings

        - effect: removes the room (from set of Rooms) and all its past Bookings (in set of Bookings)

    - editBooking(booking: Booking, newTitle: String, newStartTime: Time, newEndTime: Time)

        - requires: The new time slot from newStartTime to newEndTime must not conflict with any other existing bookings for that room.

        - effect: modifies booking with newTitle, newStartTime, and newEndTime


## **Address Verification**
- **concept:** AddressVerification[User, Address]
- **purpose:** To confirm a user’s link to a physical location by verifying that the mailing address they provide matches the authoritative address on record.
- **principle:** A service provider holds an authoritative address on file for a user. Later, when the user performs a transaction with a merchant, the merchant requests that the user provide an address (or part of it, like a ZIP code). This provided address is submitted to the service provider, who checks it against the address on file and returns a verification result (e.g., "match" or "no match") to the merchant
- **state:**
    - a set of RegisteredAddresses with:
        - an owner: User
        - an address: Address
- **actions:**
    - setAddress(user:User, address:Address):
        - effect: creates an RegisteredAddresses associating the user with the passed in address.

    - requestVerification(user:User, providedAddress:Address): Boolean
        - requires: an RegisteredAddresses exists for the user

        - effect: returns true if the providedAddress matches the address of the user and false otherwise

    - updateAddress(user: User, newAddress: Address):
        - requires: a RegisteredAddress exists for user

        - effect: updates the RegisteredAddress of user to set address to the newAddress passed in
