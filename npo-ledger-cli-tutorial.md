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

Many accounting tutorial materials will note that Loans, accounts receivable
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
`Expense:`.

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


Proper Documentation For Accounts
---------------------------------

Ledger CLI offers a flexible structure of tagging any entry, including
separate tags for parts of a split transaction.  This system uses those tags
to ensure proper documentation is included for each financial transaction
that occurs for the organization.

### Tags Used In This System

A list of tags can be found in the file `accounts/config/config-tags.ledger`
in this project.

#### Receipt Tag

The `Receipt:` tag refers to receipt of some sort.  Typically, this is a
document that shows clear confirmation that the transaction has already
occurred.  The value of the `Receipt:` tag is always a valid pathname in the
repository to the document.

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
the repository to the document.

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
`Statement:` tag is always a valid pathname in the repository to the document.

Some examples of appropriate uses of the `Statement:` tag are:

* bank statements, as received from the banking institution.

* written reports of travel.

* blog posts made by a contractor documenting their work.

* written organizational policies about the expense.

* just about anything that is clearly not an [invoice](invoice-tag) nor a
  [receipt](receipt-tag), but definitely is valid backup documentation for
  the transaction.

### Expense Account Documentation

Each Expense account entries need to be tagged with an
[`Invoice`](#invoice-tag), [`Receipt`](#receipt-tag), or
[`Statement`](#statement-tag) tag.  The value of the tag is a relative path
name of a file elsewhere in the same repository that documents the specific
expense.  For example, an entry like this:

     2012-02-05 Office Supply Galore - Online Order
         Expense:Main Org:Office Supplies      $35.00
             ;Receipt: accounts/documentation/org/receipts/2012-02-05_office-supply-galore.txt
         Liabilities:Credit Card:Visa         -$35.00

shows that a purchase was made at Office Supply Galore's online store for
$35.00, and the file `accounts/documentation/org/receipts/2012-02-05_office-supply-galore.txt`
contains the receipt from that purchase.

#### payee with "NEVER CHARGED"

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

