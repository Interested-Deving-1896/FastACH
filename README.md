[update-readmes]   Mode: rewrite — migrating to template structure...
# FastACH

[![Built with Ona](https://ona.com/build-with-ona.svg)](https://app.ona.com/#https://github.com/Interested-Deving-1896/FastACH)

<!-- AI:start:what-it-does -->
_Description pending._
<!-- AI:end:what-it-does -->

## Architecture

<!-- AI:start:architecture -->
_Architecture documentation pending._
<!-- AI:end:architecture -->

## Install

<!-- Add installation instructions here. This section is yours — the AI will not modify it. -->

```bash
git clone https://github.com/Interested-Deving-1896/FastACH.git
cd FastACH
```

## Usage


### Reading ACH file
``` csharp
var achFile = await AchFile.Read("ACH.txt");

achFile.WriteToConsole(); // Output to console
```

``` csharp
// Line map can be used for the error reporting
var lineMap = new List<(IRecord record, uint line)>();
var achFile = await AchFile.Read("ACH.txt", lineMap);
```

### Reading ACH file to colorful console
``` csharp
var achFile = await AchFile.Read(name);
achFile.WriteToConsole();
```

![Console Output](https://raw.githubusercontent.com/MaratFattakhov/FastACH/master/doc/read_to_console.png)

### Writing ACH file using AchFileBuilder (Recommended)

The `AchFileBuilder` provides a fluent API for building ACH files with less boilerplate code.

#### Simple Example - Credit and Debit Transactions
``` csharp
var achFile = new AchFileBuilder()
    .With(
        ImmediateDestination: "123456789",
        ImmediateOrigin: "987654321",
        ImmediateDestinationName: "Bank of America",
        ImmediateOriginName: "My Corporation")
    .WithBatch(batch => batch
        .With(
            CompanyId: "1234567890",
            OriginatingDFIID: "12345678",
            CompanyEntryDescription: "PAYROLL",
            CompanyName: "My Company")
        .WithCreditTransaction(
            amount: 1500.00m,
            routingNumber: "111111111",
            accountNumber: "987654321",
            receiverName: "John Doe")
        .WithDebitTransaction(
            amount: 500.00m,
            routingNumber: "222222222",
            accountNumber: "123456789",
            receiverName: "Jane Smith"))
    .Build();

await achFile.WriteToFile("ACH.txt");
```

#### Advanced Example - With All Optional Fields
``` csharp
var achFile = new AchFileBuilder()
    .With(
        ImmediateDestination: "123456789",
        ImmediateOrigin: "987654321",
        ImmediateDestinationName: "PNC Bank",
        ImmediateOriginName: "Microsoft Inc.",
        ReferenceCode: "REF12345",
        FileIdModifier: 'A')
    .WithBatch(batch => batch
        .With(
            CompanyId: "1234567890",
            OriginatingDFIID: "12345678",
            CompanyEntryDescription: "PAYROLL",
            CompanyName: "My Company",
            ServiceClassCode: 200,
            entryClassCode: "PPD",
            CompanyDiscretionaryData: "DISCRETIONARY",
            CompanyDescriptiveDate: new DateOnly(2024, 1, 15),
            EffectiveEntryDate: new DateOnly(2024, 1, 31),
            OriginatorsStatusCode: '1',
            BatchNumber: 1)
        .WithCreditTransaction(
            amount: 1500.00m,
            routingNumber: "111111111",
            accountNumber: "987654321",
            receiverName: "Employee One",
            receiverId: "EMP001",
            discretionaryData: "PAY")
        .WithAddenda(
            addendaTypeCode: 5,
            addendaInformation: "Salary payment for January 2024",
            addendaSequenceNumber: 1))
    .Build();

await achFile.WriteToFile("ACH.txt");
```

#### Multiple Batches Example
``` csharp
var achFile = new AchFileBuilder()
    .With("123456789", "987654321")
    .WithBatch(batch => batch
        .With("1234567890", "12345678", "PAYROLL", "Company A")
        .WithCreditTransaction(1000.00m, "111111111", "ACCT001", "Employee 1")
        .WithCreditTransaction(1200.00m, "222222222", "ACCT002", "Employee 2"))
    .WithBatch(batch => batch
        .With("0987654321", "87654321", "INVOICE", "Company B")
        .WithDebitTransaction(500.00m, "333333333", "ACCT003", "Customer 1")
        .WithDebitTransaction(750.00m, "444444444", "ACCT004", "Customer 2"))
    .Build();

await achFile.WriteToFile("ACH.txt");
```

### Writing ACH file (Manual Approach)
``` csharp
var achFile = new AchFile()
{
    FileHeader = new FileHeaderRecord()
    {
        ImmediateDestination = "123456789",
        ImmediateOrigin = "123456789",
        FileCreationDate = DateOnly.FromDateTime(DateTime.Now),
        FileCreationTime = TimeOnly.FromDateTime(DateTime.Now),
        FileIdModifier = 'A',
        ImmediateDestinationName = "PNC Bank",
        ImmediateOriginName = "Microsoft Inc.",
        ReferenceCode = "00000000"
    },
    BatchRecordList =
    {
        new BatchRecord()
        {
            BatchHeader = new BatchHeaderRecord()
            {
                ServiceClassCode = 200,
                CompanyName = "companyName",
                CompanyDiscretionaryData = "companyDiscretionary",
                CompanyId = "companyID",
                CompanyEntryDescription = "EntryDescr",
                CompanyDescriptiveDate = new DateOnly(2011, 02, 03),
                EffectiveEntryDate = new DateOnly(2011, 01, 02),
                OriginatingDFIID = "DFINumber"
            },
            TransactionRecords =
            {
                new TransactionRecord
                {
                    EntryDetail = new EntryDetailRecord()
                    {
                        TransactionCode = 22,
                        ReceivingDFIID = 12345678,
                        CheckDigit = '9',
                        DFIAccountNumber = "1313131313",
                        Amount = 22M,
                        ReceiverIdentificationNumber = "ID Number",
                        ReceiverName = "ID Name",
                        DiscretionaryData = "Desc Data",
                        AddendaRecordIndicator = true,
                    },
                    AddendaRecords = new List<AddendaRecord>
                    {
                        new AddendaRecord()
                        {
                            AddendaInformation = "Monthly bill"
                        }
                    }
                },
                new TransactionRecord()
                {
                    EntryDetail = new EntryDetailRecord()
                    {
                        TransactionCode = 27,
                        ReceivingDFIID = 12345678,
                        CheckDigit = '9',
                        DFIAccountNumber = "1313131313",
                        Amount = 27M,
                        ReceiverIdentificationNumber = "ID Number",
                        ReceiverName = "ID Name",
                        DiscretionaryData = "Desc Data",
                        AddendaRecordIndicator = false,
                    }
                }
            }
        }
    }
};

await achFile.WriteToFile("ACH.txt");
```

## Configuration

<!-- Document configuration options here. This section is yours — the AI will not modify it. -->

## CI

<!-- AI:start:ci -->
_CI documentation pending._
<!-- AI:end:ci -->

## Mirror chain

<!-- AI:start:mirror-chain -->
This repo is maintained in [`Interested-Deving-1896/FastACH`](https://github.com/Interested-Deving-1896/FastACH) and mirrored through:

```
Interested-Deving-1896/FastACH  ──►  OpenOS-Project-OSP/FastACH  ──►  OpenOS-Project-Ecosystem-OOC/FastACH
```

Changes flow downstream automatically via the hourly mirror chain in
[`fork-sync-all`](https://github.com/Interested-Deving-1896/fork-sync-all).
Direct commits to OSP or OOC are detected and opened as PRs back to `Interested-Deving-1896`.
<!-- AI:end:mirror-chain -->

## Contributors

<!-- AI:start:contributors -->
_Contributors pending._
<!-- AI:end:contributors -->

## Origins

<!-- AI:start:origins -->
_Original project — no upstream fork._
<!-- AI:end:origins -->

## Resources

<!-- AI:start:resources -->
_No additional resource files found._
<!-- AI:end:resources -->

## License

<!-- AI:start:license -->
[MIT](https://github.com/Interested-Deving-1896/FastACH/blob/master/LICENSE) © 2026 [Interested-Deving-1896](https://github.com/Interested-Deving-1896)
<!-- AI:end:license -->
