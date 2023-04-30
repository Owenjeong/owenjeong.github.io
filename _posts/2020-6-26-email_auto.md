---
layout: single
title: Email Automation(VBA)
categories: Coding
toc: false
toc_sticky: true
toc_label: Contents
author_profile: false
---

Did you get tired of sending a same email to a bunch of people? 

Here is the key that you might be able to put less efforts to do it.

If you are sales man or payroll person or any of you who is related to this will be happy with it.



```vb
Option Explicit

Sub AutoMail()
Application.ScreenUpdating = False
Dim OutApp As Outlook.Application
Dim OutMail As Outlook.MailItem
Dim OutAccount As Outlook.Account
'Dim OutNS As Outlook.Namespace

Set OutApp = CreateObject("Outlook.Application")
'Set OutMail = OutApp.CreateItem(olMailItem)
'Set OutAccount = OutApp.Session.Accounts.Item(2)
    'OutMail.SendUsingAccount = OutNS.Accounts.Item(1)

'Or us the name instead of the number
    'Set OutAccount = OutApp.Session.Accounts("ron@something.nl")

'Creating variable to hold values of different items of mail
Dim sendTo, subj, atchmnt1, atchmnt2, msg, ccTo, bccTo As String

Dim lstRow As Long

'My data is on sheet "Exceltip.com" you can have any sheet name.
 
ActiveWorkbook.Sheets("Mail").Activate
'Getting last row of containing email id in column 3.
lstRow = Cells(Rows.Count, 4).End(xlUp).Row

'Variable to hold all email ids

Dim rng, cell As Range
Set rng = Range("D2:D" & lstRow)

'initializing outlook object to access its features
Set OutApp = New Outlook.Application
On Error GoTo cleanup 'to handle any error during creation of object.

'Loop to iterate through each row, hold data in of email in variables and send
'mail to each email id.


For Each cell In rng
    sendTo = Range(cell.Address).Offset(0, 0).Value2
    subj = Range(cell.Address).Offset(0, 1).Value2
    msg = Range(cell.Address).Offset(0, 2).Value2
    atchmnt1 = Range(cell.Address).Offset(0, -2).Value2
    atchmnt2 = Range(cell.Address).Offset(0, -1).Value2
    ccTo = Range(cell.Address).Offset(0, 3).Value2
    bccTo = Range(cell.Address).Offset(0, 4).Value2

    On Error Resume Next 'to hand any error during creation of below object
    Set OutMail = OutApp.CreateItem(0)
    
    'Writing and sending mail in new mail
    With OutMail
        .To = sendTo
        .cc = ccTo
        .BCC = bccTo
        .Body = msg
        .Subject = subj
        .Attachments.Add atchmnt1
        .Attachments.Add atchmnt2
        .Display      'this send mail without any notification. If you want see mai
        '.Send         'before send, usey .Display method.
         
    
    End With
    On Error GoTo 0 'To clean any error captured earlier
    Set OutMail = Nothing 'nullifying outmail object for next mail
 Next cell 'loop ends

cleanup: 'freeing all objects created
        Set OutApp = Nothing
        Application.ScreenUpdating = True
        Set OutAccount = Nothing
Application.ScreenUpdating = True
End Sub
```