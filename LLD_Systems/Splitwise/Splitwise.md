# Splitwise LLD

Payment Management Application:
- Application to manage payments done by multiple people in a group and simplify them. 
- Provide an ability to split the expence amoung users.

4 friends - 
Ram Shyam Nishant and Gargi --> Went to ice cream parlour

Purchased cone.

Payment was done by RAM - 400

Ram Shyam Nishant Gargi
300 -100. -100    -100                                            -- Equal Share


Went to Dmart and Purchased different thing.
                 RAM      Shyam      Nishant      Gargi
Cost of good.    10       50          35           55.    ==> 150
Paid by          150
Net Amt          +140      -50        -35          -55            --UnEqual Share [EXACT]


                RAM.     Shyam       Nishant      Gargi
                60%       10%         20%          10%
Paid By         400
Net.            +160     -40          -80          -40            --Percentage Share



A & B went to restaurent for lunch.
A paid 600
B = -300 unsettled

B needs to pay A 300rs.   -[1st Transaction]

C & B went on a dinner
B paid here 600
C = -300.         Balance: Any amount can be represented by two things: Currency and number.

C needs to pay 300 to B.  -[2nd Trasaction]

How we can minimize the transactions?

C can give directly to A -- Everything will be settled in 1 transaction









Problem: Design a payment tracking application that helps manage and track shared expences, both invidualy and in group.

Requirements:

Functional:
    You went to a Goa trip.
    One person paid for hotel accomodation - 10,000

    In a diary, Hotel Accomodation - Amount - Who all people are involed.

1. Expense Management
    - Add Expense : User should be able to add the expenses (10,000)
    - Edit Expense : Users can modify the expense
    - Settle Expense : Users can settle the expense to clear outstanding.

2. Group Management
    
    All the people going on a Goa trip will be added here.

    - Create Group
    - Add/Edit/Settle expences

3. Additional Features
    - Comment : Users can add comment to expenses.
    - Activity Log: Track all the events
    - Authentication



Non Functional Requirement:
- Scalability
- Performance
- Data Consistency.


Split Algorithms:

1. Equal Split 
    - Friends splitting equal
    - Total Amount + Participant List [input]
    - Equal Shares for all [output]

2. Exact Split
    - Unqual purchases
    - Total Amount + Per-Person amounts [input]
    - Custom amounts per person [output]

3. Percentage Split
    - Propotional sharing
    - Total Amount + per-person percentages [input]
    - Custom amount per person [output]


# Zero Sum Property:

SUM (all balances) = 0

Where:
    - Positive balance = recieves money
    - Negative Balance = Owes money 

Ram paid 400rs

Ram  Shyam Nishant Gargi
+300 -100. -100    -100    Net Bal.

300-100-100-100 = 0

Addition of all positives and negatives = 0


1. Equal Split ALgorithm

Dinner Party:
- Total bill = 900
- 3 friends : Ram Nishant Gargi
- Ram paid the full amount
- Each person should pay 300

EQUAL_SPLIT(totalAmount, paidBy, Participants)
    1. numPeople = Lenght(participants)
    2. perPersonShare = TotalAmount / numPeople
    3. balances = {}

    4. FOR each person in participants:
        IF person == paidby:
            balances[person] = totalAmount - perPersonShare
        
        ELSE:
            balances[person] = -perPersonShare
        
    5. VERIFY: SUM(balances.values) == 0
    6. return balances

2. EXACT SPLIT

- Total 500
- Ram purchased 200rs 
- Nishant purchased 180rs
- Gargi Purchased 120rs

Nishant Paid 500

exactAmount = { RAM = 200, Nishant = 180, Gargi = 120}

EXACT_SPLIT(totalAmount, paidBy, exactAmounts):    
    1. participants = key(exactAmount)

    2. sumOfShares = SUM (exactAmount.values())
    3. VERIFY: sumOfShares == TotalAmount
        Throw error if not equal.

    4. balances {}

    5. FOR each person in participants:
        userShare = exactAmount[Person]

        IF person == paidBy:
            balance = totalAmount - userShare
        
        ELSE:
            balances[Person] = -userShare

    6. VERIFY: SUM(balaces.values) == 0
    7. RETURN balances


3. PERCENTAGE split

 - Total = 2000
 - DAD pays 50% (1000)
 - Mom pays 30% (600)
 - You pays 20% (400)

DAD paid total 2000 

percentages = { DAD = 50%, MOM = 30%, YOU = 20% }

PERCENTAGE_SPLIT( totalAmount, paidBy, percentages):
    1. participants = key(percentages)
    2. sumofPercentage = SUM(percentages.values())

    3. VERFIY: sumOfPercentages = 100.0
       IF NOT: Throw...

    4. balances = {}

    5. FOR each person in participants:
        percentage = percentages[Person]
        userShare = (percentage / 100 ) * totalAmount

        IF person == paidBy:
            balances[person] = totalAmount - UserShare

        ELSE:
            balances[person] = -userShare
        
    6. VERIFY: SUM(balances.values()) == 0
    7. Return balances.


# DATA Model

User{
    userID uid,
    string ImageURL,
    int phonenumber,
    string bio
}

Balance {
    string currency;
    int amount;
}

Group{
    GroupID gid;
    List<User> users;
    string ImageURL;
    string tittle;
    string description;
}

// Expense - INdividual or group expenses, maps users to bal.
// Builder Pattern will be used : Expense builder.
Expense {
    expenseid eid;
    bool isSettled;
    map<user, Balance> userBalance;
    GroupID gid;  //Optional
    string tittle;
    int timestampl
    string imageURI;

}

// Strategy Design pattern should be used.
// Split Strategy. 
interfacte - SplitStrategy 
EQUAL, EXACT and Percentages should implement this.

//Splitwise Class: Facade Pattern

Facade? To manage all the services without letting client know. Client will only talk with facade.







