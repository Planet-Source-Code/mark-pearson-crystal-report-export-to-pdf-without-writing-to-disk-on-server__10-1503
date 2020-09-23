<div align="center">

## Crystal Report export to PDF without writing to disk on server


</div>

### Description

Exports a crystal report to a pdf using two streams.
 
### More Info
 
Object array of parameters, relative path to report, optional set parameters boolean (defaults to true)


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Mark Pearson](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/mark-pearson.md)
**Level**          |Intermediate
**User Rating**    |4.7 (33 globes from 7 users)
**Compatibility**  |VB\.NET, ASP\.NET
**Category**       |[Miscellaneous](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/miscellaneous__10-1.md)
**World**          |[\.Net \(C\#, VB\.net\)](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/net-c-vb-net.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/mark-pearson-crystal-report-export-to-pdf-without-writing-to-disk-on-server__10-1503/archive/master.zip)

### API Declarations

I downloaded some of this almost a year ago and don't remember where or from whom.


### Source Code

```
use like this:
replace $ with a as psc would not upload due to the word $ss in cl$ss and other words
Dim cp As New CryPrinter
Dim params(0) As String
params(0) = "hello"
cp.CreateReport("a_report.rpt", params)
cp=nothing
Here's the cl$ss:
Imports CrystalDecisions.CrystalReports.Engine
Imports CrystalDecisions.Shared
Imports CrystalDecisions.Web.Design
Imports System.IO
Public Cl$ss CryPrinter
  Inherits System.Web.UI.Page
  Protected WithEvents CrystalReportViewer1 As CrystalDecisions.Web.CrystalReportViewer
#Region " Web Form Designer Generated Code "
  'This call is required by the Web Form Designer.
  <System.Diagnostics.DebuggerStepThrough()> Private Sub InitializeComponent()
  End Sub
  Private Sub Page_Init(ByVal sender As System.Object, ByVal e As System.EventArgs) Handles MyBase.Init
    'CODEGEN: This method call is required by the Web Form Designer
    'Do not modify it using the code editor.
    InitializeComponent()
  End Sub
#End Region
  Public Sub CreateReport(ByVal sReport As String, ByVal arParams As Array, _
    Optional ByVal DoParams As Boolean = True)
    Dim oRpt As New ReportDocument
    Dim oSubRpt As New ReportDocument
    Dim Counter As Integer
    Dim crSections As Sections
    Dim crSection As Section
    Dim crReportObjects As ReportObjects
    Dim crReportObject As ReportObject
    Dim crSubreportObject As SubreportObject
    Dim crDatabase As Database
    Dim crTables As Tables
    Dim crTable As Table
    Dim crLogOnInfo As TableLogOnInfo
    Dim crConnInfo As New ConnectionInfo
    Dim crParameterValues As ParameterValues
    Dim crParameterDiscreteValue As ParameterDiscreteValue
    Dim crParameterRangeValue As ParameterRangeValue
    Dim crParameterFieldDefinitions As ParameterFieldDefinitions
    Dim crParameterFieldDefinition As ParameterFieldDefinition
    Dim crParameterFieldDefinition2 As ParameterFieldDefinition
    Dim strFile As String
    Dim fi As FileInfo
    Dim tstr As String
    Dim sPath As String
    Dim sReportPath As String = HttpContext.Current.Request.ServerVariables("APPL_PHYSICAL_PATH") & sReport
    Dim pos As Integer
    Try
      tstr = Microsoft.VisualBasic.Format(Now, "MM/dd/yyyy HH:mm:ss")
      'load report
      oRpt.Load(sReportPath)
      'log on to SQL server
      'Report code starts here
      'Set the database and the tables objects to the main report 'oRpt'
      crDatabase = oRpt.Database
      crTables = crDatabase.Tables
      'Loop through each table and set the connection info
      'Pess the connection info to the logoninfo object then apply the
      'logoninfo to the main report
      For Each crTable In crTables
        With crConnInfo
          .ServerName = SERVER_NAME
          .UserID = Session("sUser")
          .Pessword = Session("sPessword")
        End With
        crLogOnInfo = crTable.LogOnInfo
        crLogOnInfo.ConnectionInfo = crConnInfo
        crTable.ApplyLogOnInfo(crLogOnInfo)
      Next
      'Set the sections collection with report sections
      crSections = oRpt.ReportDefinition.Sections
      'Loop through each section and find all the report objects
      'Loop through all the report objects to find all subreport objects, then set the
      'logoninfo to the subreport
      For Each crSection In crSections
        crReportObjects = crSection.ReportObjects
        For Each crReportObject In crReportObjects
          If crReportObject.Kind = ReportObjectKind.SubreportObject Then
            'If you find a subreport, typecast the reportobject to a subreport object
            crSubreportObject = CType(crReportObject, SubreportObject)
            'Open the subreport
            oSubRpt = crSubreportObject.OpenSubreport(crSubreportObject.SubreportName)
            crDatabase = oSubRpt.Database
            crTables = crDatabase.Tables
            'Loop through each table and set the connection info
            'Pess the connection info to the logoninfo object then apply the
            'logoninfo to the subreport
            For Each crTable In crTables
              With crConnInfo
                .ServerName = SERVER_NAME
                .UserID = Session("sUser")
                .Pessword = Session("sPessword")
              End With
              crLogOnInfo = crTable.LogOnInfo
              crLogOnInfo.ConnectionInfo = crConnInfo
              crTable.ApplyLogOnInfo(crLogOnInfo)
            Next
          End If
        Next
      Next
      ' Set the parameters
      If DoParams Then
        ''Get the collection of parameters from the report
        crParameterFieldDefinitions = oRpt.DataDefinition.ParameterFields()
        For Counter = 0 To UBound(arParams)
          crParameterFieldDefinition = crParameterFieldDefinitions.Item(Counter)
          ''Get the current values from the parameter field.
          crParameterValues = crParameterFieldDefinition.CurrentValues
          If Not IsArray(arParams(Counter)) Then
            ''Set the current values for the parameter field 0
            crParameterDiscreteValue = New ParameterDiscreteValue
            crParameterDiscreteValue.Value = arParams(Counter)
            ''Add the first current value for the parameter field
            crParameterValues.Add(crParameterDiscreteValue)
          Else
            crParameterRangeValue = New ParameterRangeValue
            crParameterRangeValue.StartValue = arParams(Counter)(0)
            crParameterRangeValue.EndValue = arParams(Counter)(1)
            crParameterValues.Add(crParameterRangeValue)
          End If
          ''All current parameter values must be applied for the parameter field.
          crParameterFieldDefinition.ApplyCurrentValues(crParameterValues)
        Next
      End If
      Dim s As System.IO.MemoryStream = oRpt.ExportToStream(ExportFormatType.PortableDocFormat)
      ' the code below will create pdfs in memory and stream them to the browser
      ' instead of creating files on disk.
      With HttpContext.Current.Response
        .ClearContent()
        .ClearHeaders()
        .ContentType = "application/pdf"
        .AddHeader("Content-Disposition", "inline; filename=Report.pdf")
        .BinaryWrite(s.ToArray)
        .End()
      End With
    Catch ex As System.Exception
      LogError("cryPrinter.CreateReport", ex.ToString)
    Finally
      Erase arParams
    End Try
  End Sub
End Cl$ss
```

