# VB.NET---ChooseFromList
How to create Choose From List component using VB.NET
Hi friends, today i will explain you about 'how to create CFL(Choose From List) in VB.NET'

# Using Here
1. Visual Studio
2. SQL Server Reporting Service(SSRS) / SQL Server Management Studio (SSMS) - DBMS

# Here,
> Now I create new project  name as 'ChooseFromList'
> Create two forms, 1 for our own design and another for CFL design

# Steps
1. Design your own Form
2. Design a CFL Form
3. Call the CFL Form by load the data and show it in your own Form

We will see the above steps in detail
Okay, here we needs two forms are our own form and CFL form
So i created two forms

# 1. Design your own form
> This is my form design (4 textboxs and 5 labels for item name,code,unit,price).
> When i click item text box, The CFL will load and appears below.
> Then the CFL selection will be filled in Item , Code, Unit and Price automatically.
> In Here, I define 1 event and its code for load the CFL by query and fill the form data

Private Sub txtitem_Click(sender As Object, e As EventArgs) Handles txtitem.Click
	Dim xleft As Integer = Me.Location.X + txtitem.Location.X + 20 ' X postion of CFL Form
        Dim ytop As Integer = Me.Location.Y + txtitem.Location.Y + txtitem.Height + 35 ' Y Postion
        CFL.FunLoadCFL("select * from item", xleft, ytop) ' Call the CFL Form by sending query
        If Not IsNothing(CFL.ColumnValue) Then		  ' Check If any data is selected
            txtitem.Text = CFL.ColumnValue(0)             ' Fill the first cell data in item box
            txtcode.Text = CFL.ColumnValue(1)		  ' Fill the next cell data in code box
            txtprice.Text = CFL.ColumnValue(2)		  ' Fill the next cell data in price box
            txtunit.Text = CFL.ColumnValue(3)		  ' Fill the next cell data in unit box
        End If
End Sub


The FunLoadCFL() function and ColumnValue() array variable is belongs to CFL Form
some of the above code will be cleared below. continue watching

# 2. Design a CFL form And Open it as a dialog
> This is my CFL Form Design (1 label, 1 textbox, 1 data gridview)
> The data will be loaded in data grid view. When you search data in textbox, it will selected in it.
> In here, I define 3 functions for CFL form, 2 function for DB connection & Data selection 
and 3 events
# Function for CFL form
1. FunLoadCFL() --> Used to loads  CFL form, data in grid view and fixed the grid view
2. FunSearch() --> Used to select the searched data in textbox on data grid view.
3. FunSaveSelection() --> Used to save the selected row values in array variable and close it.
# Function for DB connection and selection
4. FunConnectDB() --> used to connect DB
5. FunFillDT() --> used to select the data from database table and filled it with .NET datatable
# Events
1. textbox_search_TextChanged() --> callFunSearch() function
2. CFL_KeyDown() --> close the CFL Form when escape key is pressed
3. dgvload_CellClick() --> when select in data grid view, call the function FunSaveSelection()


# The User Defined Function are below
Public Sub FunLoadCFL(ByVal str_query As String, ByVal left As Integer, ByVal Top As Integer,
Optional ByVal frm_Width As Integer = 400, Optional ByVal frm_Height As Integer = 200)
        Me.StartPosition = FormStartPosition.Manual   'Say Location
        Me.DesktopLocation = New Point(left, Top)     'Locate the CFL in My Location
        Me.Width = frm_Width : Me.Height = frm_Height 'Set Heigth and Width optional
        Me.KeyPreview = True                          'Set keypreview for keydown event here
        dgvload.AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill 'Fix the column in grid
        dgvload.SelectionMode = DataGridViewSelectionMode.FullRowSelect    'Select full row 
        dgvload.AllowUserToAddRows = False            'Don't allow user to add the grid
        dgvload.AllowUserToDeleteRows = False	      'Don't allow user to delete the grid
        dt = New DataTable                            'define datatable
        dt = FunFillDT(str_query)		      'Fill data in data table
        If dt.Rows.Count > 0 Then		      'If no row in datatable don't load the grid    
            dgvload.DataSource = dt		      'load the data grid with datatable
            dgvload.Columns(0).DefaultCellStyle.Font = New Font(FontFamily.GenericSansSerif, 9, FontStyle.Bold)'Set Font style for column bold and big
        End If
        txtsearch.Text = ""			      'onload set the search text box empty
        Me.ShowDialog()				      'Show/Open it as a dialog window
End Sub
Private Sub FunSearch(ByVal searchtext As String)
        Try
            If txtsearch.Text.Trim() <> "" Then		'If textbox search is not empty allow
                For i As Integer = 0 To dgvload.RowCount - 1 'search the data in gridview
	            Dim SData as String = Char.ToUpper(dgvload.Rows(i).Cells(0).Value.ToString)
        	    If SData = Char.ToUpper (txtsearch.Text) Then 'If data in grid and textbox is same
                        dgvload.Rows(i).Selected = True		  'select the row in grid
                        dgvload.CurrentCell = dgvload.Item(0, i)  'set current cell in grid
                    End If
                Next
            End If
        Catch ex As Exception

        End Try
End Sub
Private Sub FunSaveSelection()
        Try
            ReDim ColumnValue(dgvload.ColumnCount)   'Re_Declare the array size as grid column count
            For i As Integer = 0 To dgvload.ColumnCount - 1 'Loop to columncount for save the data
	       Dim CD as String = dgvload.Rows(dgvload.CurrentCell.RowIndex).Cells(i).Value.ToString()
               ColumnValue(i) = CD      'Save selected row data cell by cell & one by one
            Next
            Me.DialogResult = System.Windows.Forms.DialogResult.OK
            Me.Close() ' close the dialog
        Catch ex As Exception

        End Try
End Sub    
Public Function FunFillDT(ByVal str_query As String) As Object
        Try
            If IsNothing(con) Then
                FunConnectDB()
            End If
            dt = New DataTable
            da = New SqlDataAdapter(str_query, con)
            da.Fill(dt)
        Catch ex As Exception
            MsgBox(ex.ToString())
        End Try
        Return dt
End Function
Public Sub FunConnectDB()
        Try
            If Not IsNothing(con) Then
                con = Nothing
            End If
            con = New SqlConnection("DATA SOURCE = 'DESKTOP-074SEAB'; INTEGRATED SECURITY = false; INITIAL CATALOG = 'Items'; USER ID ='sa'; PASSWORD ='sql@123';")
	    'server name : DESKTOP-074SEAB, DB Name : Items, DBUser : sa, DBPassword : sql@123
            con.Open()
            con.Close()
        Catch ex As Exception
            MsgBox("ex.ToString(), , "SQL CONNCETION ERROR")
        End Try
End Sub



# The Events are Below
Private Sub txtsearch_TextChanged(sender As Object, e As EventArgs) Handles txtsearch.TextChanged
        FunSearch(txtsearch.Text.Trim())
End Sub

Private Sub CFL_KeyDown(sender As Object, e As KeyEventArgs) Handles Me.KeyDown
        If e.KeyCode = Keys.Escape Then
            Me.DialogResult = Windows.Forms.DialogResult.Cancel
            Me.Close()
        End If
End Sub
Private Sub dgvload_CellClick(sender As Object, e As DataGridViewCellEventArgs) Handles dgvload.CellClick
        FunSaveSelection()
End Sub

# Code for MyForm
Dim xleft As Integer = Me.Location.X + txtitem.Location.X + 20
Dim ytop As Integer = Me.Location.Y + txtitem.Location.Y + txtitem.Height + 35
CFL.FunLoadCFL("select * from item", xleft, ytop)
If Not IsNothing(CFL.ColumnValue) Then
    txtitem.Text = CFL.ColumnValue(0)
    txtcode.Text = CFL.ColumnValue(1)
    txtprice.Text = CFL.ColumnValue(2)
    txtunit.Text = CFL.ColumnValue(3)
End If
# Code for CFL Form
#Region "Imports"
    Imports System.Windows.Forms
    Imports System.Data.SqlClient
#End Region
#Region "Declaration"
    Public con As SqlConnection
    Public da As SqlDataAdapter
    Public dt As DataTable
    Public ColumnValue() As String
#End Region
#Region "User Function"
    Public Sub FunLoadCFL(ByVal str_query As String, ByVal left As Integer, ByVal Top As Integer, Optional ByVal frm_Width As Integer = 400, Optional ByVal frm_Height As Integer = 200)
        Me.StartPosition = FormStartPosition.Manual
        Me.DesktopLocation = New Point(left, Top)
        Me.Width = frm_Width : Me.Height = frm_Height
        Me.KeyPreview = True
        dgvload.AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill
        dgvload.SelectionMode = DataGridViewSelectionMode.FullRowSelect
        dgvload.AllowUserToAddRows = False
        dgvload.AllowUserToDeleteRows = False

        dt = New DataTable
        dt = FunFillDT(str_query)
        If dt.Rows.Count > 0 Then
            dgvload.DataSource = Nothing
            dgvload.DataSource = dt
            dgvload.Columns(0).DefaultCellStyle.Font = New Font(FontFamily.GenericSansSerif, 9, FontStyle.Bold)
            dt1 = dt.Copy()
        End If
        txtsearch.Text = ""
        Me.ShowDialog()
    End Sub
    Private Sub FunSearch(ByVal searchtext As String)
        Try
            If txtsearch.Text.Trim() <> "" Then
                For i As Integer = 0 To dgvload.RowCount - 1
                    If Char.ToUpper(dgvload.Rows(i).Cells(0).Value.ToString) = Char.ToUpper(txtsearch.Text) Then
                        dgvload.Rows(i).Selected = True
                        dgvload.CurrentCell = dgvload.Item(0, i)
                    End If
                Next
            Else
                dgvload.DataSource = dt1
            End If
        Catch ex As Exception

        End Try
    End Sub
    Private Sub FunSaveSelection()
        Try
            ReDim ColumnValue(dgvload.ColumnCount)
            For i As Integer = 0 To dgvload.ColumnCount - 1
                ColumnValue(i) = dgvload.Rows(dgvload.CurrentCell.RowIndex).Cells(i).Value.ToString()
            Next
            Me.DialogResult = System.Windows.Forms.DialogResult.Cancel
            Me.Close()
        Catch ex As Exception

        End Try
    End Sub
    Public Function FunFillDT(ByVal str_query As String) As Object
        Try
            If IsNothing(con) Then
                FunConnectDB()
            End If
            dt = New DataTable
            da = New SqlDataAdapter(str_query, con)
            da.Fill(dt)
        Catch ex As Exception
            MsgBox(ex.ToString())
        End Try
        Return dt
    End Function
    Sub FunConnectDB()
        Try
            If Not IsNothing(con) Then
                con = Nothing
            End If
            con = New SqlConnection("DATA SOURCE = 'DESKTOP-074SEAB'; INTEGRATED SECURITY = false; INITIAL CATALOG = 'Items'; USER ID ='sa'; PASSWORD ='sql@123';")
            con.Open()
            con.Close()
        Catch ex As Exception
            MsgBox("Connection error Check service  : " + vbNewLine + ex.ToString(), , "SQL CONNCETION ERROR")
        End Try
    End Sub
#End Region
