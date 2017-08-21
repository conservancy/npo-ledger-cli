Non-Profit Accounting With Ledger CLI, A Tutorial
=================================================

Non-profit organizations (NPOs), particularly 501(c)(3) charities in the USA,
have their own specific accounting needs.  These often differ from for-profit
accounting needs.  For example, for-profit-oriented systems often make
problematic assumptions about the workflow of accounting tasks (often because
NPOs rely primarily on donations, rather than fee-for-service or
widget-selling income).  Also, non-profit income is categorized differently
than for-profit income, and the reporting requirements vary wildly from their
for-profit equivalents.

This project is designed to provide some basic templates, tutorials, workflow
documentation and scripts to handle accounting for an NPO.  The primary
example is a
[direct project (aka Model A) fiscal sponsor NPO](http://en.wikipedia.org/wiki/Fiscal_sponsorship#Models_of_fiscal_sponsorship).

This tutorial was written primarily based on
[Software Freedom Conservancy](http://sfconservancy.org/)'s use of Ledger CLI
from 2008-10-22 to present for its own accounting needs.  While Conservancy
has done well using this system, and believes that its account system meets
Generally accepted accounting principles (GAAP), this document **does not**
constitute advice from a CPA nor legal advice for a non-profit that seeks to
comply with relevant state and/or federal accounting requirements for USA
non-profits.  The authors make no representations nor warranties regarding
this information and this information is provided for discussion purposes
only.  Readers of these tutorial and templates are urged to seek professional
advice from a CPA and/or tax legal counsel in constructing an accounting
system appropriate for your organization.

Furthermore, given the authors' limited knowledge of accounting requirements
outside the USA, the suggestions herein probably are not particularly useful
at all for organizations outside the USA.

Configuration of Chart of Accounts
----------------------------------

The first thing any accountant will ask to see if your so-called "chart of
accounts".  The first time I heard this phrase, I thought it was something
complicated.  Fact of the matter is, it's really just a list of all the
accounts that you use.  Accountants also use "account codes", which, as near
as I can tell, are of primary interest because they get better sorting.
Ledger CLI doesn't really support account codes, so I've ignored them.

The real place that Ledger CLI stores your chart of accounts is if you use
the `account` directive along with the `--pedantic` CLI option.  This will
ensure that only accounts you declared explicitly will used.

### Asset Accounts

Asset accounts represent anything that's owned.  Typically, these are
primarily your cash accounts, or anything that's completely liquid.

Many accounting tutorial materials will note that loans, accounts receivable
and other receivables are assets as well.  Most accountants will
say that they are, but with regard to accounts called "Assets", this system
uses the account hierarchy `Assets:` only for tangible, liquid,
cash and/or cash-equivalent assets.  You'll find that account hierarchy
commonly in the examples herein.

### Liabilities Accounts

Similar to assets, most accountants will point out that any amount owed to
someone else is a liability, and that is of course accurate.  Like with the
`Assets:` hierarchy, this system uses `Liabilities:` hierarchy only to refer
to formalized accounts, such as credit cards, where a monthly statement is
sent and have an ongoing liability relationship with the organization.

### Accrued Accounts

For items that are receivable or payable, this system uses `Accrued:`
hierarchy.  Under this top-level account, you'll find accounts payable,
accounts receivable, loans payable and loans receivable.

### Expense Accounts

These accounts contain any expense of the organization, and all begin with
`Expenses:`.

### Income Accounts

These accounts contain any income of the organization, and all begin with
`Income:`.

### Unearned Income Accounts

`Unearned Income:` accounts are used to refer to revenue that is currently
received for services which have not yet been delivered.  The most typical
and common place an NPO encounters this type of income is for conference
registrations.  Since conference registrations arrive in advance of the
conference, it is not proper under accrual accounting to call it income until
such time as the conference successfully completes.

### Reporting The Chart of Accounts

The
[`general-ledger-report.plx` script in the `non-profit-audit-reports` Ledger CLI contrib directory](https://github.com/ledger/ledger/blob/next/contrib/non-profit-audit-reports/general-ledger-report.plx)
will generate a file called `chart-of-accounts.csv`, which is the chart of accounts.

The main command-line program though, that generates the chart of accounts
looks like this:
    $ ledger -f accounts/main/books.ledger -V -F "%-150A\n" -w -s -b 2012/01/01 -e 2013/01/01 reg

Note that this is bound by date.  Typically, it makes sense to list your
chart of accounts for a specific period (e.g., your fiscal year), since your
accounts might have some cruft in them from previous years that should now be
ignored.  (For example, if your organization simplified its chart of accounts
in later years, you don't want to report those old accounts that are no
longer used.)

Handling Fiscal Sponsorship
---------------------------

NPOs that do not provide fiscal sponsorship services will find this section
somewhat useless.  One of the biggest benefits of Ledger CLI is its
incredible flexibility that just does not exist in other accounting systems.
This section describes how to exploit that flexibility to provide a
separation in your books and reporting to handle earmarked accounts for
fiscally sponsored projects.

NPOs that don't need this feature can, in most cases, use the methods
described herein to deploy Ledger CLI, but should leave out the `:General:`
and `:ProjectNAME:` parts of the account hierarchy, since these are the
primary mechanisms used herein to handle the fiscal sponsorship structure.

### Earmarked Accounts

Many fiscal sponsor NPOs keep earmarked accounts for their member/affiliated
projects.  Furthermore, these projects often may either (a) terminate their
agreement with the NPO, and thus deserve a copy of their books that they can
"take away" with them, or (b) might be affiliated with *other* NPOs that also
hold accounts.  This system of earmarked accounts is designed to make it easy
for projects to have a copy of their own accounts, but not interfere with nor
even be aware of (a) the books of other member/affiliated projects, and (b)
the overall books of the entire NPO.

On the latter point, this system utilizes a directory structure and separate
`.ledger` files to separate out the different projects into different
structures.  This allows member/affiliated projects to take their data and
run `ledger` commands against it, separately and without access to the other
`.ledger` files of the NPO.


Proper Documentation For All Transactions
-----------------------------------------

Ledger CLI offers a flexible structure of tagging any entry, including
separate tags for parts of a split transaction.  This system uses those tags
to ensure proper documentation is included for each financial transaction
that occurs for the organization.

Note that since Ledger CLI is a complete double-entry accounting system, each
transaction can correspond to multiple entries in the general ledger.  The
data entry format of Ledger CLI lists each double-entry accounting
transaction in a text file.

Documentation may in fact differ for entries within the transaction.  Ledger
CLI's tagging structure is flexible in this regard: each portion of a
double-entry transaction can carry the same tag or a different tag.  For
example, in this entry:

    2012-05-03 Sir Moneybags
            ;Entity: Sir-Moneybags
            ;Invoice: accounts/documentation/org/invoices/2012-05-30_moneybags-invoice_as-sent.txt
        Accrued:Accounts Receivable:Conservancy  $100,000.00
        Income:Main Org:Donations               $-100,000.00
            ;IncomeType: Donations

The portion of the transaction that credits the `Income:Main Org:Donations`
has three tags: [`Entity`](#entity-tag), [`Invoice`](#invoice-tag) and
[`IncomeType`](#income-type).  The `Entity` and `Invoice` tags, since they're
listed at the top of the transaction, propagate through and apply to both
sides.  But, the `IncomeType` tag, which has no meaning for `Accrued:`
accounts, so it is applied only to the `Income:Main Org:Donations` part of
the transaction.

Below you'll find detailed descriptions of all the possible tags that are
used in this system.  The actual declarations and enforcement of rules of
these tags can be found in the file `accounts/config/config-tags.ledger` in
this project.


### Documentation Tags

Documentation tags are tags that link to other backup documents that provide
evidence and details that justify a particular ledger entry.  The value of
the tag is a relative path name of a file elsewhere in the same repository
that documents the specific expense.  For example, an entry like this:

     2012-02-05 Office Supply Galore - Online Order
         Expenses:Main Org:Office Supplies     $35.00
             ;Receipt: accounts/documentation/org/receipts/2012-02-05_office-supply-galore.txt
         Liabilities:Credit Card:Visa         -$35.00

shows that a purchase was made at Office Supply Galore's online store for
$35.00, and the file
`accounts/documentation/org/receipts/2012-02-05_office-supply-galore.txt`
contains the receipt from that purchase.

#### Receipt Tag

The `Receipt:` tag refers to receipt of some sort.  Typically, this is a
document that shows clear confirmation that the transaction has already
occurred.  The value of the `Receipt:` tag is always a valid pathname in the
repository to the document, [as described above](#documentation-tags).

Some examples of appropriate uses of the `Receipt:` are:

* a point-of-sale credit card receipt from a purchase, given by a cashier or
  sent via email after the purchase has occurred.

* a deposit slip given at the bank upon making an over-the-counter deposit of
  paper checks.

* a confirmation document showing an outgoing wire transfer made by a bank.

* a confirmation document showing transfer of funds between two bank
  accounts.

* A pay advice document generated upon payment of an invoice.

#### Invoice Tag

The `Invoice:` tag refers to an actual invoice, either generated by the
organization or received by the organization.  Typically, this is a document
that is a request for payment, rather than documenting an actual payment that
has occurred.  The value of the `Invoice:` tag is always a valid pathname in
repository to the document, [as described above](#documentation-tags).

Some examples of appropriate uses of the `Invoice:` tag are:

* an actual invoice as sent by a vendor to the organization.

* a request for payment sent by the organization to someone else.

* a reimbursement request submitted by an employee, contractor, or volunteer
  for expenses they've already incurred and would like the organization to
  reimburse (e.g., an expense report, requesting for reimbursement of travel
  expenses).

#### Statement Tag

The `Statement:` tag refers to any sort of written statement received from an
external party (or even perhaps generated internally) that provides document,
insight, or other information about the transaction.  The value of the
`Statement:` tag is always a valid pathname in the repository to the
document, [as described above](#documentation-tags).


Some examples of appropriate uses of the `Statement:` tag are:

* bank statements, as received from the banking institution.

* written reports of travel.

* blog posts made by a contractor documenting their work.

* written organizational policies about the expense.

* just about anything that is clearly not an [invoice](#invoice-tag) nor a
  [receipt](#receipt-tag), but definitely is valid backup documentation for
  the transaction.

#### TaxReporting Tag

The `TaxReporting` tag is an optional tag for `Assets` accounts that debit to
the account.

When provided, the `TaxReporting` accompanies a `TaxImplication` information
tag.  The TaxReporting refers to a document that verifies the choice for the
`TaxImplication` tag.  For example, for individual contractors in the USA, a
`TaxImplication` of `1099` would be well served by a `TaxImplication` that
links to a [W-9](https://www.irs.gov/pub/irs-pdf/fw9.pdf) for the individual
being paid.  For a individual foreign contractor, the `TaxReporting` might
link to a
[W8-BEN](https://www.irs.gov/uac/form-w-8ben-certificate-of-foreign-status-of-beneficial-owner-for-united-states-tax-withholding)
for the payee.

### Information Tags

In contrast to documentation tags, information tags can more traditionally be
considered pure "meta-data" for a ledger entry.

#### Entity Tag

The `Entity:` tag is required for many types of ledger entries.  The value of
the `Entity:` tag is a unique moniker that identifies the organization,
company, person, or legal entity that is the external party for the
transaction.

Note that there is no database of these monikers, so typos can cause
trouble.  However, you could implement checks in
`accounts/config/config-tags.ledger` using a regular expression to verify no
typos have occurred.  This would be somewhat cumbersome, since Ledger CLI
would likely require that the monikers be encoded into a regular expression.
Barring that, the
[integrity of your data should be periodically checked](checking-integrity-of-tag).

#### IncomeType Tag

The `IncomeType:` tag is used for all `Income:` accounts.  This refers to the
type of income.  The value of the `IncomeType:` tag is always a string.
Since this particular system is designed for USA non-profit entities who file
USA Form 990, the following `IncomeType` values are supported:

* `Donations`, which refers to standard charitable donations.

* `RBI`, which refers to "related business income".

* `UBTI`, which refers to "unrelated business taxable income.

Not that donor-advised funds and government grants don't currently have their
own `IncomeType`.  It's possible this might be necessary; the authors aren't
familiar with how to handle those items on the Form 990.  It would be a
relatively simple change to `config-tags.ledger`, though, to support other
income types, or to change it entirely to handle use-cases other than USA
Form 990 filing.

#### TaxImplication Tag

The `TaxImplication:` tag is used for all `Asset:` accounts when the
transaction includes a payment of $10.00 or more leaving the account.  This
tag catalogs any tax implications that might occur on outgoing funds.

The most important USA-related issue tracked by this tag are contractors who
must have annual 1099 and/or W2 issued.  An [`Entity:` tag](entity-tag) should always
go along with a TaxImplication tag.

The possible values for this field are:

* `1099`, indicating the amount paid requires issuance of a USA Federal Form
  1099 for the `Entity` involved.

* `W2`, indicating the amount paid will be part of a USA Federal income, as
  reported on Form W2 report for the `Entity` involved.

* `Retirement-Pretax`, indicating the amount paid was made to a W2 employee
  as part of pre-tax retirement plan, such as a 401(k) or 403(b) plan.

* `Accountant-Advises-No-1099`, indicating that the circumstances and rules
  seem to indicate a USA Federal Form 1099 should be issued for the `Entity`
  involved, but an outside accountant advised that no 1099 need be issues for
  this `Entity`.

* `Bank-Transfer`, indicating that the amount is a transfer between two
  banking accounts under the control of the NPO itself.

* `Foreign-Individual-Contractor`, indicating that the NPO has established
  that the `Entity` is a contractor residing outside the USA who is not a USA
  citizen and does not for any reason pay taxes in the USA.

* `Foreign-Corporation`, indicating that the NPO has established
  that the `Entity` is a corporation outside the USA.

* `USA-Corporation`, indicating that the NPO has established that the
  `Entity` is an incorporated entity the USA (i.e., "Inc."), and therefore no
  1099 is required.

* `USA-501c3`, indicating that the NPO has established that the `Entity`
  has federal 501(c)(3) status in the USA, and therefore no 1099 is required.

* `Refund`, indicating that the amount is a refund owed to the `Entity` from
  an amount previously paid to the NPO.

* `Reimbursement`, indicating that the amount is a reimbursement of expenses
  incurred by the `Entity` and thus it is not income to the `Entity`.

* `Tax-Payment`, indicating this is a tax payment to a taxing authority (such
  as the state or federal government) (e.g., a unrelated business income tax
  payment).

* `USA-LLC-No-1099`, indicating that the `Entity` is an LLC, but not the type
  of LLC for which the USA requires issuing a 1099.

* `Loan`, indicating that the `Entity` is receiving these funds as a loan
  that is expected to be paid back.

#### Program Tag

The `Program` tag is used primarily to track program activity for `Income:`
and `Expenses:` accounts.  This allows for knowing what particular initiative
initiated the income (e.g., a specific fundraising campaign) and/or what
particular program activity an expense is toward (e.g., funding travel to
some specific conference).

The Program tag is always a string with the same format as a Ledger CLI
account (primarily for use with Ledger CLI's `--pivot` and `--group-by`,
[as described later](#testing-program-success)).

#### GrantLocation Tag

The `GrantLocation` tag is used to indicate that an expense is a grant.  The
value for the tag should indicate the geographical region.  It is recommend
that the geographical reason be identified with the
[ISO 3166-2](https://en.wikipedia.org/wiki/ISO_3166-2) two letter country
code for the country where the grant goes.

This tag is to assist in filing
Form 990, [Schedule I](https://www.irs.gov/uac/about-schedule-i-form-990) and
[Schedule F](https://www.irs.gov/charities-non-profits/exempt-organizations-annual-reporting-requirements-foreign-activities-form-990-schedule-f-activities-reported).

### Account Type Documentation Requirements

Each account type has different documentation requirements.  Based on the
type of the account, it requires a different set of tags.

When Ledger CLI's `--pedantic` option is used, these rules are enforced by
ledger itself via the configurations found in `config-tags.ledger` and
`config-accounts.ledger`.

#### Expense Account Documentation

Each `Expenses:` account entry must be tagged with the following tags:

* One of: [`Invoice:`](#invoice-tag) [`Receipt:`](#receipt-tag), or
  [`Statement`](#statement-tag).  (The only exception to this rule: an entry
  does not need an `Invoice:`, `Receipt`, nor a `Statement` tag if the
  [payee was never charged](#never-charged-payee).)

* A [`Program:`](#program-tag) tag.

Expense accounts can have the following optional tag:

* A [`GrantLocation:`](#grantlocation-tag) tag.

#### NEVER CHARGED Payee

The only exception to the standard tagging requirement is when the payee has
been modified to indicate that the expense was `NEVER CHARGED`.  This is an
historical special-case.  The solution was originally design for the
following scenario:

Suppose an expense was expected &mdash; for example, a situation where you
gave a credit card number to charge something and the charge never came
through &mdash; but it turns out the charge never happened.

The recommended way to resolve this problem in the system is to just delete
the entry entirely from the Ledger file, and allow the VCS to log the fact
that the charge was expected, but the vendor never billed the credit card.

The reason the `NEVER CHARGED` payee text was added was to handle the
situation where the books included this charge, but the books were already
closed for the financial period (e.g., the books had already been audited).
Changing the payee was a method for documenting the expense.  You might use
it like this:

    2011/05/28 My Bad Billing Hosting - NEVER CHARGED
        Liabilities:Credit Card:Visa            $-100.00
        Expenses:Conservancy:Hosting             $100.00

    2012/01/01 My Bad Billing Hosting - REVERSAL - NEVER CHARGED
        Liabilities:Credit Card:Visa             $100.00
        Expenses:Conservancy:Hosting            $-100.00

However, going forward, you'd likely never enter anything the ledger
**until** you had real proof via an Invoice, Receipt or Statement that showed
the Expense did/should occur.  This use of `NEVER CHARGED` in the payee is
thus deprecated.

#### Income Account Documentation

Each `Income:` account must have the following tags:

* One of: [`Invoice:`](#invoice-tag),
  [`PurchaseOrder:`](#purchase-order-tag),
  [`Statement:`](#statement-tag) or
  [`Contract`](#contract-tag).  Exceptions to this requirement are as follows:
     + the income generated from the transaction is less than $800, or
     + the `IncomeType` is `RBI` and the income is for a defined, public
       program (such as conference registration)

* An [`Entity:`](#entity-tag) tag, *iff.* the Income for the transaction is
  for more than $800.

* An [`IncomeType:`](#incometype-tag) tag.

* A [`Program:`](#program-tag) tag.

Reports For Various Situations
------------------------------

When data is well formed as specified herein, there are various quick reports
that can be used to verify certain conditions or issues with accounting.
Here are a few examples:

### Verifying That An Invoice Is Paid

Assume for this example that the shell variables `entity`, `program`, and
`invoice` are set as follows:

    $ entity=Sir-Moneybags; program='Main.*Org:.*Direct'; invoice=2012-05-30

If the invoice was paid, this ledger command will have two lines of output,
and the second line will be a transaction on the payment date.  If only one
line appears, it's the receivable accrual and we see the invoice is not paid.

    $ ledger -f accounts/books.ledger  -V --sort d --limit 'tag("Entity") =~ /'$entity'/ and tag("Program") =~ /'$program'/ and tag("Invoice") =~ /'$invoice'/' reg /Accrued/

Alternatively, the following command will only have output if the invoice is unpaid:

    $ ledger -f accounts/books.ledger  -V --limit 'tag("Entity") =~ /'$entity'/ and tag("Program") =~ /'$program'/ and tag("Invoice") =~ /'$invoice'/' bal /Accrued/

### Calculating Donation Portion of Invoice

As described in
[IRS 501(c)(3) rules](https://www.irs.gov/charities-non-profits/substantiating-charitable-contributions),
a charity must substantiate portions of a contribution that are RBI, and
portions that are donations separately and inform the donor which portion was
a donation that may be eligible for tax deduction.  Use of the `IncomeType`
tag facilities collection of the necessary data to determine the total, and
the following command line will total up all the donation portions for a
particular invoice:

    $ entity=BigCorp; program=Foo.*Conf.*2017; invoice=2016-01-30
    $ ledger -f accounts/books.ledger  -V --sort d --limit 'tag("IncomeType") =~ /Donation/ and tag("Entity") =~ /'$entity'/ and tag("Program") =~ /'$program'/ and tag("Invoice") =~ /'$invoice'/' reg

Analysis of the Data
--------------------

If this methodology is followed, Ledger can be used to analyze the financial
data for the organization.

### Testing Program Success

If you use the [`Program`](#program-tag) tag effectively, you can easily test
the successes of various fundraising programs with a command like this:

    $ ledger -f accounts/books.ledger --pivot Program bal '/^Income/'

Meanwhile, using the  [`Program`](#program-tag) tag for Expenses can help
track what programs are costing with commands like these:

    $ ledger -f accounts/books.ledger --group-by 'tag("Program")' reg '/^Expenses/'

FIXME: example output

### Checking Integrity of a Tag

[As mentioned](#entity-tag), the `Entity:` tag is one example among many
where the value is a wide range, but since Ledger CLI isn't backed by a more
complete ERP system, it's possible during data entry for typos to make a
serious problem.  One work around to this flaw is to periodically run a
command like:

    $ ledger -f accounts/books.ledger -F '%(tag("Entity"))\n' reg|sort|uniq|less

which will show all unique `Entity:` values currently in use.

Copyright and License of This File
----------------------------------

This specific document, the README.md file for npo-ledger-cli, is copyrighted:
  Copyright &copy; 2013, Bradley M. Kuhn

This document's license gives you freedom; you can copy, modify, convey,
propagate, and/or redistribute this software under the terms of either:

    * The GNU General Public License as published by the Free Software
      Foundation, Inc.; either version 3 of the License, or (at your option)
      any later version (aka GPLv3-or-later).

    * *or* the Creative Commons Attribution-ShareAlike 3.0 United States
      license, as published by Creative Commons, Inc. (aka CC-By-SA-USA-3.0)

In addition, when you convey, distribute, and/or propagate this document
and/or modified versions thereof, you may also preserve this notice so that
recipients of such distributions will also have both licensing options
described above.

A copy of GPLv3 and CC-By-SA-3.0-USA can be found in the same repository as
this file under the filenames GPLv3.txt and CC-By-SA-3.0-USA.txt.  If this
document has been separated from the repository, a
[copy of GPL can be found on FSF's website](http://www.gnu.org/licenses/gpl.txt)
and a
[copy of CC-By-SA-USA-3.0 can be found on Creative Commons' website](http://creativecommons.org/licenses/by-sa/3.0/us/legalcode).

