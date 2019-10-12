# MinsaPay Anonymized Transaction Data 2019

[![](https://img.shields.io/badge/license-mit-orange?longCache=true&style=for-the-badge)](LICENSE.md) [![](https://img.shields.io/badge/read_in-한국어-blue?longCache=true&style=for-the-badge)](README-KO.md)

MinsaPay is an electronic payment system designed for the KMLA Summer Festival (Minjok Festival). The MinsaPay team has decided to open the entire transaction data to help students studying data analysis and organizing future Minjok Festivals. Because financial data that can specify individual users are susceptible to data abusing, the MinsaPay team has anonymized the transaction data. Every other aspect of the data has not been modified.

## Statistics

|Content|Data|
|----|----|
|Users|400|
|Transactions|2,900|
|Aggregate amount|Transacted $14,400 approx.|

## Index and explanations

|Index|Explanation|
|----|----|
|transaction|Indicates transaction number|
|user|Indicates the anonymized user number|
|booth|Indicates the anonymized booth number|
|timestamp|Indicates the time of the transaction|
|type|Indicates the transaction type|
|amount|Indicates the amount of transaction|
|balance|Indicates the user's balance after the transaction|

There are 4 types of transactions

|Type|Explanation|
|----|----|
|open|Opening the payment account. Opening the payment account. We provide ₩7,000 (about \$7) of subsidy for senior students and ₩10,000 (about \$10) of subsidy for teachers. Please check additional info.|
|charge|Charging the payment account. The school council member will process the request and deposit the cash.|
|pay|Paying the bills.|
|withdraw|Withdrawing the prepaid money (usually after the festival). The school council member will process the request and deliver the cash.|

Booths can calculate the sales statement afterward. Every profits other than the production cost will be used for student council operation fees.

## Additional info

* Accounts were opened and charged at the student council booth, which is `booth#000`.
* Festival organizers have agreed not to charge our teachers. Therefore, teachers were charged more than ₩1,000,000 (about \$1,000)
* There were three students with charge errors.
    * user#298 charged ₩5,464,526 (about \$4,550) at transaction 561. This was a human error made by the student council member entering random values in the input field. This was fixed 12 seconds later by the same member, at transaction 565.
    * user#295 charged ₩2,147,483,647 (about \$1,787,440) at transaction 557. This was a human error by the student council member tagging the Student RFID in the charge amount field. Since the RFID value was greater than the max value that int can represent, it was converted to the max value that an int can represent. The student council has later recognized this problem and fixed at transaction 2624 by manually calculating the student’s actual balance.
    * user#284 charged ₩2,147,483,647 (about \$1,787,440) at transaction 2844. This was the same type of human error like the one of user#295 and was fixed 11 seconds later at transaction 2845.
* At the day of the festival, the MinsaPay server went down for `13 minutes 15 seconds` from 10:17:55 AM to 10:31:10 AM. This can be seen by the gap between transaction `1546` and `1547`. The problem was due to the sudden surge of user request exceeding the free tier limit of the database. The problem was fixed soon, and the problem did not reoccur after purchasing a paid plan.
* During the momentary server failure, each booths created their own billing spreadsheet. After the festival, these spreadsheets were collected and merged. Students payed their unpaid bills through the credit payment booth, which is booth#017.

## `anonymize.ipynb`

```python
import pandas as pd
Dataframe = pd.read_csv('raw.csv')
def anonymize(df, targetColumn):
    anon = {}
    id = 0
    for x in range(len(df)):
        user = df.loc[x, targetColumn]
        if user in anon:
            df.loc[x, targetColumn] = anon[user]
        else:
            if id < 10:
                unknown = "#00"+ str(id)
            elif id < 100:
                unknown = "#0" + str(id)
            else:
                unknown = "#"    + str(id)
            anon[user] = targetColumn + str(unknown)
            id += 1
            df.loc[x, targetColumn] = anon[user]
anonymize(Dataframe, 'user')
anonymize(Dataframe, 'booth')
Dataframe.to_csv("anonymized.csv", mode='w')
```
* Each user and booths were numbered as they appear.
* Same values are anonymized with the same number.
* See the [anonymized list of transactions](transactions.csv).
