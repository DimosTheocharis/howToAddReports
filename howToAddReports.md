
### Λογική

Κάθε φορά που θέλουμε να φέρουμε δεδομένα από την βάση τα οποία δεν βρίσκονται όλα μαζί σε ένα συγκεκριμένο μοναδικό table αλλά προκύπτουν από περίπλοκη σύνδεση πολλών tables οφείλουμε να ακολουθήσουμε την παρακάτω διαδικασία η οποία αποκαλείται *adding reports*. Για την κατανόηση της διαδικασίας συμπεριλαμβάνεται το παράδειγμα για την εισαγωγή των *leaves rights*.

### Βήματα

##### 1. Εισαγωγή του query στο service του backend

Στο βήμα αυτό απαιτείται η υλοποίηση μιας συνάρτησης η οποία φέρνει τα δεδομένα από το database σύμφωνα με το query. Για παράδειγμα στο αρχείο `employeeleavesright.service.ts` υλοποιήθηκε η:


```
    public async getLeavesReportData():Promise<LeavesReportDto[]> | null {
        return await this.employeeLeavesRightRepository.manager.query(`
          SELECT
            EmployeeLeavesRight.EmployeeLeavesRightID,
            Employees.FirstName,
            Employees.LastName,
            Leaves.Description,
            EmployeeLeavesRight.EmployeeLeavesRightMaxDays,
            EmployeeLeavesRight.EmployeeLeavesRightYear,
            Employees.EmployeeArithmos
          FROM
            EmployeeLeavesRight
            INNER JOIN  Employees ON Employees.EmployeeID=EmployeeLeavesRight.EmployeeID
            INNER JOIN  Leaves ON Leaves.LeaveID=EmployeeLeavesRight.LeaveID
          WHERE
            Employees.EmployeeActive = 1
          `
          )
        .then(data => data as LeavesReportDto[]);
      }
```

To service που περιέχει την συνάρτηση αυτή βρίσκεται στο path: *Source\prosopikoapi\src\datalayer\services*

##### 2. Απορρόφηση των δεδομένων στο controller του backend

Το συγκεκριμένο βήμα συνιστά την δημιουργία μιας συνάρτησης στο controller η οποία καλεί την συνάρτηση του service. Για παράδειγμα στο αρχείο `employeeleavesright.controller.ts` υλοποιήθηκε η:

```
    async getLeavesReport():Promise<LeavesReportDto[]> | null{
        return this.service.getLeavesReportData();
    }
```

Ο διαχωρισμός της λογικής από το controller είναι σημαντικός για να κρατάμε το αρχείο αυτό καθαρό και συμπαγές.
Η συνάρτηση θα πρέπει να συνδεθεί με μία api get μέθοδο με αντιπροσωπευτικό όνομα, όπως:
```
    @Get("reportdata")
```
Αυτό θα αποτελεί το *endpoint* στο οποίο θα 'χτυπάμε' για να αντλήσουμε τα δεδομένα του report.

Το controller μπορεί να βρεθεί στο path: *Source\prosopikoapi\src\api\controllers*


##### 3. Καθορισμός της δομής των δεδομένων στο model του backend

Τα δεδομένα που έρχονται από το database και διαμοιράζονται από το service στο controller χρειάζεται να έχουν συγκεκριμενη δομή. Συνεπώς απαιτείται ο σαφής ορισμός του μοντέλου των δεδομένων (πεδία και τύπος δεδομένων). Για παράδειγμα στο αρχείο `LeavesReportDto.ts` υλοποιήθηκε το μοντέλο: 

```
    export class LeavesReportDto {
        @ApiProperty()
        employeeLeavesRightID: number;
        @ApiProperty()
        firstName: string;
        @ApiProperty()
        lastName: string;
        @ApiProperty()
        description: string;
        @ApiProperty()
        employeeLeavesRightMaxDays: number;
        @ApiProperty()
        employeeLeavesRightYear: number;
        @ApiProperty()
        employeeArithmos: number;
      }
```

Το model μπορεί να βρεθεί στο path: *Source\prosopikoapi\src\api\models*


##### 4. Σύνδεση frontend με backend στο service του frontend

Η μεταφορά των δεδομένων στο frontend προυποθέτει την υλοποίηση μιας συνάρτησης στο service η οποία εκτελεί ένα http get request στο *endpoint* με το οποίο συνδέθηκε η συνάρτηση στο controller. Για παράδειγμα στο αρχείο `employee-leaves-right.service.ts` υλοποιήθηκε η:

```
    public getLeavesReport(): Observable<LeavesReport[]> | null{
        const url = `${AppSettings.API_ENDPOINT}employeeleavesright/reportdata`;
        return this.httpClient.get(url, this.httpOptions)
            .pipe(
                map(data => {
                    return data || [];
                }),
                tap(console.log),
                catchError(error => {
                    this.handleError(error);
                    throw error;
                })
            );
    }
```

Το service μπορεί να βρεθεί στο path: *Source\prosopikoapp\src\app\shared\services*






    


