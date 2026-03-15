# 🗂️ DotNet Developer Notes — Master Structure v2

> **Stack:** C# · ASP.NET Core MVC · ADO.NET · SQL Server · JavaScript · AJAX · Kendo UI
> **Goal:** Interview-ready reference + long-term study plan
> **DSA:** 1200+ LC problems solved — structured as quick-recall notes, not tutorials

---

## 📐 HOW TO READ THIS

```
DotNet-Developer-Notes/
├── 01_CSharp/                    ← subject  (numbered = learning order)
│   ├── A.Fundamentals/           ← category (lettered)
│   │   ├── 01_Overview.md        ← chapter  (numbered .md file)
│   │   └── 02_Program_Structure.md
│   └── E.Collections/            ← some categories are folders-of-folders
│       ├── 01_Arrays/            ← sub-category folder
│       │   └── 01_Array_Basics.md
│       └── 09_Generics/
│           └── 01_Generics_Overview.md
```

---

---

## 01_CSharp/

> OOP lives **inside** C# — after methods. You learn the language mechanics first,
> then immediately apply them through OOP. No separate OOP subject.

```
01_CSharp/
│
├── A.Fundamentals/
│   ├── 01_CLR_and_NET_Overview.md
│   ├── 02_Program_Structure.md
│   ├── 03_Variables_and_Constants.md
│   ├── 04_Data_Types.md
│   ├── 05_Value_vs_Reference_Types.md
│   ├── 06_Type_Conversion_and_Casting.md
│   ├── 07_Nullable_Types.md
│   ├── 08_Operators.md
│   └── 09_Control_Statements.md
│
├── B.Methods/
│   ├── 01_Methods_Basics.md
│   ├── 02_Parameters_and_Arguments.md
│   ├── 03_Optional_and_Named_Parameters.md
│   ├── 04_ref_out_in_Keywords.md
│   ├── 05_Method_Overloading.md
│   └── 06_Recursion.md
│
├── C.OOP/                              ← RIGHT after methods — where it belongs
│   │
│   ├── 01_Classes_and_Objects/
│   │   ├── 01_Class_vs_Object.md
│   │   ├── 02_Fields_Properties_Methods.md
│   │   └── 03_this_Keyword.md
│   │
│   ├── 02_Encapsulation/
│   │   ├── 01_Encapsulation_Concept.md
│   │   ├── 02_Access_Modifiers.md
│   │   ├── 03_Properties_Getters_Setters.md
│   │   └── 04_Data_Hiding.md
│   │
│   ├── 03_Constructors/
│   │   ├── 01_Default_Constructor.md
│   │   ├── 02_Parameterized_Constructor.md
│   │   ├── 03_Copy_Constructor.md
│   │   ├── 04_Static_Constructor.md
│   │   └── 05_Constructor_Chaining.md
│   │
│   ├── 04_Inheritance/
│   │   ├── 01_Inheritance_Basics.md
│   │   ├── 02_Base_and_Derived_Class.md
│   │   ├── 03_Method_Hiding_vs_Overriding.md
│   │   ├── 04_base_Keyword.md
│   │   └── 05_Constructor_in_Inheritance.md
│   │
│   ├── 05_Polymorphism/
│   │   ├── 01_Compile_Time_Method_Overloading.md
│   │   ├── 02_Runtime_Method_Overriding.md
│   │   ├── 03_virtual_override_new_Keywords.md
│   │   └── 04_Upcasting_and_Downcasting.md
│   │
│   ├── 06_Abstraction/
│   │   ├── 01_Abstract_Classes.md
│   │   ├── 02_Interfaces.md
│   │   ├── 03_Abstract_Class_vs_Interface.md      ← top interview question
│   │   └── 04_Explicit_Interface_Implementation.md
│   │
│   ├── 07_Object_Relationships/
│   │   ├── 01_Association.md
│   │   ├── 02_Aggregation.md
│   │   ├── 03_Composition.md
│   │   └── 04_HAS_A_vs_IS_A.md
│   │
│   ├── 08_Advanced_OOP/
│   │   ├── 01_Static_Classes_and_Members.md
│   │   ├── 02_Sealed_Classes.md
│   │   ├── 03_Partial_Classes.md
│   │   └── 04_Object_Class_Methods.md             ← ToString, Equals, GetHashCode
│   │
│   └── 09_SOLID_Principles/
│       ├── 01_Single_Responsibility.md
│       ├── 02_Open_Closed.md
│       ├── 03_Liskov_Substitution.md
│       ├── 04_Interface_Segregation.md
│       ├── 05_Dependency_Inversion.md
│       └── 06_SOLID_in_Controller_BAL_DAL.md      ← real project example
│
├── D.Language_Features/
│   ├── 01_Delegates.md
│   ├── 02_Events.md
│   ├── 03_Lambda_Expressions.md
│   ├── 04_Extension_Methods.md
│   ├── 05_Anonymous_Types.md
│   ├── 06_Tuples_and_ValueTuple.md
│   ├── 07_Records.md
│   └── 08_Pattern_Matching.md
│
├── E.Collections/                      ← deep, sub-categorized
│   │
│   ├── 00_Collection_Interfaces.md     ← IEnumerable, ICollection, IList, IDictionary
│   │
│   ├── 01_Arrays/
│   │   ├── 01_Array_Basics.md
│   │   ├── 02_Multidimensional_Arrays.md
│   │   ├── 03_Jagged_Arrays.md
│   │   └── 04_Array_Class_Methods.md
│   │
│   ├── 02_List/
│   │   ├── 01_List_T_Basics.md
│   │   ├── 02_List_Methods.md
│   │   └── 03_List_vs_Array.md
│   │
│   ├── 03_Dictionary/
│   │   ├── 01_Dictionary_TKey_TValue.md
│   │   ├── 02_SortedDictionary.md
│   │   ├── 03_ConcurrentDictionary.md
│   │   └── 04_Dictionary_Patterns.md
│   │
│   ├── 04_Sets/
│   │   ├── 01_HashSet_T.md
│   │   └── 02_SortedSet_T.md
│   │
│   ├── 05_Queue_and_Stack/
│   │   ├── 01_Queue_T.md
│   │   ├── 02_Stack_T.md
│   │   └── 03_PriorityQueue.md
│   │
│   ├── 06_LinkedList/
│   │   └── 01_LinkedList_T.md
│   │
│   ├── 07_Strings/
│   │   ├── 01_String_Basics.md
│   │   ├── 02_String_Methods.md
│   │   ├── 03_StringBuilder.md
│   │   └── 04_String_Interning_and_Immutability.md
│   │
│   └── 08_Generics/
│       ├── 01_Generics_Overview.md
│       ├── 02_Generic_Classes.md
│       ├── 03_Generic_Methods.md
│       ├── 04_Generic_Constraints.md
│       └── 05_Covariance_and_Contravariance.md
│
├── F.LINQ/
│   ├── 01_LINQ_Overview.md
│   ├── 02_Query_Syntax.md
│   ├── 03_Method_Syntax.md
│   ├── 04_Filtering_Projection_Ordering.md
│   ├── 05_Grouping_and_Joining.md
│   └── 06_Aggregates_and_Set_Operations.md
│
├── G.Modern_CSharp/
│   ├── 01_Null_Conditional_and_Coalescing.md
│   ├── 02_String_Interpolation.md
│   ├── 03_Expression_Bodied_Members.md
│   ├── 04_Deconstruction.md
│   └── 05_Spread_and_Indices.md
│
├── H.Exception_Handling/
│   ├── 01_try_catch_finally.md
│   ├── 02_Exception_Types_Hierarchy.md
│   └── 03_Custom_Exceptions.md
│
├── I.Async_and_Concurrency/
│   ├── 01_Threads_Overview.md
│   ├── 02_Task_and_Task_T.md
│   ├── 03_Async_Await.md
│   ├── 04_ConfigureAwait.md
│   └── 05_CancellationToken.md
│
└── J.Memory_and_Runtime/
    ├── 01_Garbage_Collection.md
    ├── 02_IDisposable_and_using.md        ← critical for ADO.NET connections
    ├── 03_Stack_vs_Heap.md
    └── 04_Boxing_and_Unboxing.md
```

---

## 02_ASP_NET_Core_MVC/

```
02_ASP_NET_Core_MVC/
│
├── A.Fundamentals/
│   ├── 01_What_is_ASP_NET_Core.md
│   ├── 02_ASP_NET_Core_vs_Framework.md
│   ├── 03_MVC_Architecture.md
│   ├── 04_Request_Processing_Pipeline.md
│   ├── 05_Program_cs_and_Startup.md
│   └── 06_Project_Structure.md
│
├── B.MVC_Pattern/
│   ├── 01_Model_Responsibilities.md
│   ├── 02_View_Responsibilities.md
│   ├── 03_Controller_Responsibilities.md
│   └── 04_MVC_Request_Flow.md
│
├── C.Controllers/
│   ├── 01_Creating_Controllers.md
│   ├── 02_Action_Methods.md
│   ├── 03_IActionResult_vs_ActionResult_T.md
│   ├── 04_Action_Results_Types.md
│   ├── 05_Model_Binding_in_Controllers.md
│   └── 06_Controller_Base_vs_ControllerBase.md
│
├── D.Views/
│   ├── 01_Razor_View_Engine.md
│   ├── 02_Razor_Syntax.md
│   ├── 03_Layout_Pages.md
│   ├── 04_Partial_Views.md
│   ├── 05_View_Components.md
│   └── 06_Tag_Helpers.md
│
├── E.Data_Passing/
│   ├── 01_ViewData.md
│   ├── 02_ViewBag.md
│   ├── 03_TempData.md
│   └── 04_Strongly_Typed_Views.md
│
├── F.Routing/
│   ├── 01_Conventional_Routing.md
│   ├── 02_Attribute_Routing.md
│   ├── 03_Route_Constraints.md
│   └── 04_Route_Parameters.md
│
├── G.Model_Handling/
│   ├── 01_Model_Binding.md
│   ├── 02_Model_Validation.md
│   ├── 03_Data_Annotations.md
│   └── 04_Fluent_Validation.md
│
├── H.Dependency_Injection/
│   ├── 01_DI_Overview.md
│   ├── 02_Singleton_Scoped_Transient.md
│   ├── 03_Registering_Services.md
│   └── 04_Constructor_Injection.md
│
├── I.Middleware_and_Filters/
│   ├── 01_Middleware_Concepts.md
│   ├── 02_Built_in_Middleware.md
│   ├── 03_Custom_Middleware.md
│   ├── 04_Filters_Overview.md
│   ├── 05_Action_Filters.md
│   └── 06_Exception_Filters.md
│
├── J.Web_API/
│   ├── 01_API_Controllers.md
│   ├── 02_ApiController_Attribute.md
│   ├── 03_HTTP_Verbs.md
│   ├── 04_Routing_in_Web_API.md
│   ├── 05_FromBody_FromQuery_FromRoute.md
│   ├── 06_Returning_JSON.md
│   ├── 07_Status_Codes.md
│   └── 08_Global_Exception_Handling.md
│
├── K.Security/
│   ├── 01_Authentication_Overview.md
│   ├── 02_Cookie_Authentication.md
│   ├── 03_JWT_Authentication.md
│   ├── 04_JWT_Generation_and_Validation.md
│   ├── 05_OAuth_and_External_Login.md
│   ├── 06_Authorization_Overview.md
│   ├── 07_Role_Based_Authorization.md
│   ├── 08_Policy_Based_Authorization.md
│   ├── 09_Claims_Based_Identity.md
│   └── 10_CORS.md
│
├── L.State_Management/
│   ├── 01_Cookies.md
│   ├── 02_Sessions.md
│   └── 03_Distributed_Session.md
│
├── M.Caching/
│   ├── 01_In_Memory_Cache.md
│   ├── 02_Response_Caching.md
│   └── 03_Distributed_Cache.md
│
├── N.Background_Services/
│   ├── 01_IHostedService.md
│   └── 02_BackgroundService_Base_Class.md
│
├── O.Configuration/
│   ├── 01_appsettings_json.md
│   ├── 02_Environment_Variables.md
│   ├── 03_IConfiguration_Interface.md
│   └── 04_Options_Pattern.md
│
├── P.Error_Handling/
│   ├── 01_Exception_Handling_Middleware.md
│   └── 02_Custom_Error_Pages.md
│
└── Q.Deployment/
    ├── 01_Environments_Dev_Staging_Prod.md
    ├── 02_Logging_with_ILogger.md
    ├── 03_Serilog_Setup.md
    └── 04_Hosting_and_Publishing.md
```

---

## 03_SQL/

```
03_SQL/
│
├── A.Fundamentals/
│   ├── 01_Database_Concepts.md
│   ├── 02_Relational_Database_Model.md
│   ├── 03_RDBMS_vs_NoSQL.md
│   └── 04_SQL_Server_and_SSMS_Basics.md
│
├── B.DDL/
│   ├── 01_CREATE_TABLE.md
│   ├── 02_ALTER_TABLE.md
│   ├── 03_DROP_and_TRUNCATE.md
│   └── 04_Constraints_PK_FK_UNIQUE_CHECK.md
│
├── C.DML/
│   ├── 01_INSERT.md
│   ├── 02_UPDATE.md
│   ├── 03_DELETE.md
│   └── 04_MERGE_Upsert.md
│
├── D.Query_Basics/
│   ├── 01_SELECT_Statement.md
│   ├── 02_WHERE_Filtering.md
│   ├── 03_ORDER_BY_and_TOP.md
│   ├── 04_DISTINCT.md
│   └── 05_LIKE_IN_BETWEEN_NULL.md
│
├── E.Joins/
│   ├── 01_Inner_Join.md
│   ├── 02_Left_Join.md
│   ├── 03_Right_Join.md
│   ├── 04_Full_Outer_Join.md
│   ├── 05_Cross_Join.md
│   └── 06_Self_Join.md
│
├── F.Aggregations/
│   ├── 01_Aggregate_Functions.md
│   ├── 02_GROUP_BY.md
│   ├── 03_HAVING.md
│   └── 04_ROLLUP_and_CUBE.md
│
├── G.Advanced_Querying/
│   ├── 01_Subqueries.md
│   ├── 02_Correlated_Subqueries.md
│   ├── 03_CTEs.md
│   ├── 04_Window_Functions.md            ← ROW_NUMBER, RANK, LEAD, LAG
│   └── 05_PIVOT_and_UNPIVOT.md
│
├── H.Database_Objects/
│   ├── 01_Views.md
│   ├── 02_Stored_Procedures.md
│   ├── 03_Scalar_Functions.md
│   ├── 04_Table_Valued_Functions.md
│   └── 05_Triggers.md
│
├── I.Transactions/
│   ├── 01_Transactions_Basics.md
│   ├── 02_ACID_Properties.md
│   ├── 03_COMMIT_ROLLBACK_SAVEPOINT.md
│   └── 04_Isolation_Levels.md
│
└── J.Performance/
    ├── 01_Indexes_Overview.md
    ├── 02_Clustered_vs_NonClustered.md
    ├── 03_Reading_Execution_Plan.md
    ├── 04_Query_Optimization.md
    └── 05_Normalization_1NF_2NF_3NF.md
```

---

## 04_ADO_NET/

```
04_ADO_NET/
│
├── A.Fundamentals/
│   ├── 01_What_is_ADO_NET.md
│   ├── 02_ADO_NET_Architecture.md
│   ├── 03_Data_Providers_Overview.md
│   ├── 04_Key_Namespaces.md
│   └── 05_Connection_Strings.md
│
├── B.Connected_Architecture/
│   ├── 01_Connected_vs_Disconnected.md
│   ├── 02_SqlConnection.md
│   ├── 03_SqlCommand.md
│   ├── 04_CommandType_Text_SP_Table.md
│   ├── 05_ExecuteReader.md
│   ├── 06_ExecuteScalar.md
│   ├── 07_ExecuteNonQuery.md
│   ├── 08_SqlDataReader.md
│   ├── 09_SqlParameters_Parameterized_Queries.md
│   ├── 10_Stored_Procedures_with_ADO.md
│   ├── 11_Output_Parameters.md
│   └── 12_Multiple_Result_Sets.md
│
├── C.Disconnected_Architecture/
│   ├── 01_SqlDataAdapter.md
│   ├── 02_DataSet.md
│   ├── 03_DataTable.md
│   ├── 04_DataRow_and_DataColumn.md
│   ├── 05_DataRelations.md
│   └── 06_Fill_and_Update.md
│
├── D.CRUD_Operations/
│   ├── 01_Insert_with_ADO.md
│   ├── 02_Read_with_ADO.md
│   ├── 03_Update_with_ADO.md
│   └── 04_Delete_with_ADO.md
│
├── E.Transactions/
│   ├── 01_SqlTransaction.md
│   ├── 02_BEGIN_COMMIT_ROLLBACK.md
│   └── 03_Nested_Transactions.md
│
├── F.Async_ADO_NET/
│   ├── 01_Async_Await_with_ADO.md
│   ├── 02_OpenAsync_ExecuteReaderAsync.md
│   └── 03_CancellationToken_in_ADO.md
│
├── G.Three_Layer_Architecture/           ← your real project pattern
│   ├── 01_Controller_BAL_DAL_Overview.md
│   ├── 02_DAL_Class_Design.md
│   ├── 03_BAL_Business_Logic_Design.md
│   └── 04_Passing_Models_Between_Layers.md
│
└── H.Performance_and_Config/
    ├── 01_Connection_Pooling.md
    ├── 02_Command_Timeout.md
    ├── 03_Using_Statement_Prevents_Leaks.md
    └── 04_Exception_Handling_in_ADO.md
```

---

## 05_EF_Core/

```
05_EF_Core/
│
├── A.Fundamentals/
│   ├── 01_ORM_Concept.md
│   ├── 02_EF_Core_Overview.md
│   ├── 03_EF_Core_vs_ADO_NET.md
│   └── 04_EF_Core_Architecture.md
│
├── B.DbContext/
│   ├── 01_DbContext_Overview.md
│   ├── 02_DbSet.md
│   ├── 03_Configuring_DbContext.md
│   ├── 04_Fluent_API.md
│   └── 05_Data_Annotations_in_EF.md
│
├── C.Development_Approaches/
│   ├── 01_Code_First.md
│   ├── 02_Database_First.md
│   └── 03_Choosing_an_Approach.md
│
├── D.Migrations/
│   ├── 01_Creating_Migrations.md
│   ├── 02_Applying_Migrations.md
│   ├── 03_Rolling_Back_Migrations.md
│   └── 04_Seeding_Data.md
│
├── E.CRUD_Operations/
│   ├── 01_Insert.md
│   ├── 02_Read_and_LINQ_Queries.md
│   ├── 03_Update.md
│   ├── 04_Delete.md
│   └── 05_Raw_SQL_with_EF.md
│
├── F.Relationships/
│   ├── 01_One_to_One.md
│   ├── 02_One_to_Many.md
│   └── 03_Many_to_Many.md
│
└── G.Performance/
    ├── 01_Lazy_Loading.md
    ├── 02_Eager_Loading.md
    ├── 03_Explicit_Loading.md
    ├── 04_AsNoTracking.md
    └── 05_Compiled_Queries.md
```

---

## 06_JavaScript/

> JSON is part of JavaScript — no separate subject needed.

```
06_JavaScript/
│
├── A.Fundamentals/
│   ├── 01_JavaScript_Overview.md
│   ├── 02_Variables_let_const_var.md
│   ├── 03_Data_Types.md
│   ├── 04_Type_Coercion_and_Equality.md  ← == vs ===
│   ├── 05_Operators.md
│   └── 06_Control_Statements.md
│
├── B.Functions/
│   ├── 01_Function_Declaration_vs_Expression.md
│   ├── 02_Arrow_Functions.md
│   ├── 03_this_Keyword.md                ← critical for Kendo callbacks
│   ├── 04_Closures.md
│   ├── 05_IIFE.md
│   └── 06_Callback_Functions.md
│
├── C.Objects_and_Arrays/
│   ├── 01_Objects_Creation_and_Access.md
│   ├── 02_Prototype_Chain.md
│   ├── 03_Arrays_Basics.md
│   ├── 04_Array_Methods.md
│   └── 05_Destructuring.md
│
├── D.DOM/
│   ├── 01_DOM_Structure.md
│   ├── 02_Selecting_Elements.md
│   ├── 03_DOM_Manipulation.md
│   ├── 04_Event_Handling.md
│   └── 05_Event_Bubbling_and_Delegation.md
│
├── E.Modern_JavaScript/
│   ├── 01_ES6_Features_Overview.md
│   ├── 02_Spread_and_Rest_Operators.md
│   ├── 03_Template_Literals.md
│   ├── 04_Modules_Import_Export.md
│   ├── 05_Promises.md
│   ├── 06_Async_Await.md
│   └── 07_Error_Handling.md
│
└── F.JSON/
    ├── 01_JSON_Structure.md
    ├── 02_JSON_Objects_and_Arrays.md
    ├── 03_JSON_Parse_and_Stringify.md
    ├── 04_JSON_in_ASP_NET_Core.md
    └── 05_Working_with_Nested_JSON.md
```

---

## 07_AJAX/

```
07_AJAX/
│
├── A.Fundamentals/
│   ├── 01_What_is_AJAX.md
│   ├── 02_Sync_vs_Async.md
│   └── 03_How_HTTP_Requests_Work.md
│
├── B.Implementations/
│   ├── 01_XMLHttpRequest.md
│   ├── 02_Fetch_API.md
│   └── 03_Fetch_with_Async_Await.md
│
├── C.jQuery_Deep_Dive/
│   ├── 01_jQuery_Overview.md
│   ├── 02_Selectors_and_DOM.md
│   ├── 03_DOM_Manipulation_with_jQuery.md
│   ├── 04_Events_in_jQuery.md
│   └── 05_jQuery_AJAX_Methods.md        ← $.get, $.post, $.ajax, $.getJSON
│
├── D.Practical_Usage/
│   ├── 01_AJAX_with_JSON.md
│   ├── 02_AJAX_with_ASP_NET_Web_API.md
│   ├── 03_Handling_Responses_and_Errors.md
│   ├── 04_AJAX_GET_vs_POST.md
│   ├── 05_Passing_Data_to_Controller.md
│   └── 06_Global_AJAX_Setup.md
│
└── E.Security/
    ├── 01_CSRF_Protection.md
    ├── 02_Anti_Forgery_Tokens.md
    └── 03_CORS_in_AJAX.md
```

---

## 08_Kendo_UI/

```
08_Kendo_UI/
│
├── A.Fundamentals/
│   ├── 01_Introduction_to_Kendo_UI.md
│   ├── 02_Kendo_Architecture.md
│   ├── 03_CDN_vs_Local_Setup.md
│   └── 04_Kendo_Themes.md
│
├── B.DataSource/
│   ├── 01_DataSource_Overview.md
│   ├── 02_DataSource_Transport.md
│   ├── 03_Read_Create_Update_Destroy.md
│   ├── 04_Server_Side_Paging_Sorting.md
│   └── 05_DataSource_Events.md
│
├── C.Grid/
│   ├── 01_Grid_Basics.md
│   ├── 02_Grid_Columns_Configuration.md
│   ├── 03_Grid_Editing_Modes.md         ← inline, popup, incell
│   ├── 04_Grid_Inline_Editing.md
│   ├── 05_Grid_Popup_Editing.md
│   ├── 06_Grid_Paging_Sorting_Filtering.md
│   ├── 07_Grid_Toolbar.md
│   ├── 08_Grid_Templates.md
│   ├── 09_Grid_Refresh_and_Destroy.md
│   └── 10_Grid_Events.md
│
├── D.Form_Components/
│   ├── 01_Kendo_DropDownList.md
│   ├── 02_Kendo_ComboBox.md
│   ├── 03_Kendo_DatePicker.md
│   ├── 04_Kendo_AutoComplete.md
│   ├── 05_Kendo_NumericTextBox.md
│   ├── 06_Kendo_TimePicker.md
│   └── 07_Kendo_Validation.md
│
├── E.Window_and_Dialog/
│   ├── 01_Kendo_Window.md
│   ├── 02_Kendo_Dialog.md
│   └── 03_Modal_Patterns.md
│
├── F.Charts/
│   ├── 01_Charts_Overview.md
│   ├── 02_Bar_and_Column_Charts.md
│   └── 03_Line_and_Area_Charts.md
│
└── G.Debugging_and_Patterns/
    ├── 01_Common_Kendo_Bugs.md
    ├── 02_Inspecting_API_Response_DevTools.md
    ├── 03_Grid_Not_Loading_Data.md
    └── 04_Destroy_and_Recreate_Grid.md
```

---

## 09_Git/

```
09_Git/
│
├── A.Fundamentals/
│   ├── 01_Version_Control_Concepts.md
│   ├── 02_Git_vs_Centralized_VCS.md
│   └── 03_Git_Architecture.md
│
├── B.Core_Commands/
│   ├── 01_Init_Clone_Config.md
│   ├── 02_Add_and_Commit.md
│   ├── 03_Status_Log_Diff.md
│   ├── 04_Push_and_Pull.md
│   └── 05_Fetch_vs_Pull.md
│
├── C.Branching/
│   ├── 01_Creating_and_Switching_Branches.md
│   ├── 02_Merging.md
│   ├── 03_Rebasing.md
│   └── 04_Cherry_Pick.md
│
├── D.Collaboration/
│   ├── 01_Pull_Requests.md
│   └── 02_Git_Flow_Workflow.md
│
└── E.Troubleshooting/
    ├── 01_Resolving_Merge_Conflicts.md
    ├── 02_Git_Stash.md
    └── 03_Reset_vs_Revert.md
```

---

## 10_Testing/

```
10_Testing/
│
├── A.Fundamentals/
│   ├── 01_Why_Testing_Matters.md
│   ├── 02_Unit_vs_Integration_vs_E2E.md
│   └── 03_Arrange_Act_Assert.md
│
├── B.xUnit/
│   ├── 01_xUnit_Setup.md
│   ├── 02_Writing_Test_Methods.md
│   ├── 03_Fact_and_Theory.md
│   └── 04_Assertions.md
│
├── C.Moq/
│   ├── 01_Mocking_Overview.md
│   ├── 02_Moq_Basics.md
│   ├── 03_Setup_and_Returns.md
│   └── 04_Verify_Method_Calls.md
│
└── D.Testing_ASP_NET/
    ├── 01_Testing_Controllers.md
    ├── 02_Testing_BAL_and_DAL.md
    └── 03_Integration_Testing.md
```

---

## 11_DSA/

> **You've solved 1200+ LC problems.**
> This section is structured as **pattern-first quick-recall notes** — not tutorials.
> Every data structure contains ALL its algorithms inline.
> Reading graph algorithms? Graph patterns are right there in the same folder.
> No jumping around.

---

### HOW DSA IS ORGANIZED

```
Each data structure folder contains:
  ├── 01_Basics.md          ← definition, properties, time/space, when to use
  ├── 02_Core_Operations.md ← key operations with complexity
  ├── 03_PatternName.md     ← one .md per major algorithm/pattern
  └── 0N_Interview_Patterns_Summary.md  ← quick pattern→use-case cheat sheet
```

---

```
11_DSA/
│
├── A.Complexity_Analysis/              ← read this FIRST, once
│   ├── 01_Big_O_Notation.md
│   ├── 02_Time_Complexity.md
│   ├── 03_Space_Complexity.md
│   ├── 04_Best_Average_Worst_Case.md
│   ├── 05_Amortized_Analysis.md
│   └── 06_Complexity_Cheat_Sheet.md    ← all DS operations at a glance
│
├── B.Arrays/
│   ├── 01_Array_Basics.md              ← memory layout, access O(1), insert O(n)
│   ├── 02_Two_Pointer.md               ← sorted arrays, opposite ends, same dir
│   ├── 03_Sliding_Window.md            ← fixed window, variable window patterns
│   ├── 04_Prefix_Sum.md                ← range query, 2D prefix sum
│   ├── 05_Kadanes_Algorithm.md         ← max subarray
│   ├── 06_Dutch_National_Flag.md       ← 3-way partition, sort colors
│   ├── 07_Binary_Search_on_Arrays.md   ← on sorted array, rotated array
│   ├── 08_Merge_Intervals.md           ← overlap, insert interval
│   └── 09_Array_Interview_Patterns.md  ← quick recall cheat sheet
│
├── C.Strings/
│   ├── 01_String_Basics.md             ← immutability, char array tricks
│   ├── 02_Two_Pointer_on_Strings.md    ← palindrome, reverse
│   ├── 03_Sliding_Window_on_Strings.md ← longest substring, anagram patterns
│   ├── 04_KMP_Pattern_Matching.md      ← failure function, search O(n+m)
│   ├── 05_Rabin_Karp.md                ← rolling hash
│   ├── 06_Anagram_and_Frequency_Map.md
│   └── 07_String_Interview_Patterns.md
│
├── D.Hashing/
│   ├── 01_Hash_Map_Basics.md           ← O(1) avg, collision, load factor
│   ├── 02_Frequency_Count_Pattern.md
│   ├── 03_Two_Sum_Family.md            ← complement map, k-sum variants
│   ├── 04_Subarray_Sum_with_HashMap.md ← prefix sum + map
│   └── 05_Hash_Interview_Patterns.md
│
├── E.Stack/
│   ├── 01_Stack_Basics.md              ← LIFO, O(1) push/pop
│   ├── 02_Monotonic_Stack.md           ← next greater, previous smaller
│   ├── 03_Valid_Parentheses_Family.md
│   ├── 04_Daily_Temperatures_Pattern.md
│   ├── 05_Largest_Rectangle_Histogram.md
│   └── 06_Stack_Interview_Patterns.md
│
├── F.Queue_and_Deque/
│   ├── 01_Queue_Basics.md              ← FIFO, use in BFS
│   ├── 02_Deque_Basics.md              ← double ended, O(1) both ends
│   ├── 03_Sliding_Window_Maximum.md    ← monotonic deque
│   ├── 04_Circular_Queue.md
│   └── 05_Queue_Interview_Patterns.md
│
├── G.Linked_Lists/
│   ├── 01_Linked_List_Basics.md        ← singly, doubly, O(n) access
│   ├── 02_Fast_Slow_Pointer.md         ← cycle detection, middle, nth from end
│   ├── 03_Reverse_Linked_List.md       ← iterative, recursive, in k-groups
│   ├── 04_Merge_Two_Sorted_Lists.md
│   ├── 05_LRU_Cache.md                 ← HashMap + DoublyLinkedList
│   └── 06_LL_Interview_Patterns.md
│
├── H.Recursion_and_Backtracking/
│   ├── 01_Recursion_Mental_Model.md    ← trust the recursion, base case
│   ├── 02_Backtracking_Template.md     ← choose, explore, unchoose
│   ├── 03_Subsets.md
│   ├── 04_Permutations.md
│   ├── 05_Combinations.md
│   ├── 06_N_Queens.md
│   ├── 07_Sudoku_Solver.md
│   └── 08_Backtracking_Interview_Patterns.md
│
├── I.Binary_Search/
│   ├── 01_Binary_Search_Basics.md      ← left, right, find exact
│   ├── 02_Binary_Search_on_Answer.md   ← min/max feasibility pattern
│   ├── 03_Rotated_Array.md
│   ├── 04_Search_in_2D_Matrix.md
│   ├── 05_Find_Peak_Element.md
│   └── 06_BS_Interview_Patterns.md
│
├── J.Sorting/
│   ├── 01_Sorting_Overview_Complexity.md ← all algos at a glance
│   ├── 02_Merge_Sort.md                ← divide and conquer, stable
│   ├── 03_Quick_Sort.md                ← partition, pivot, unstable
│   ├── 04_Heap_Sort.md
│   ├── 05_Counting_and_Radix_Sort.md   ← O(n) non-comparison sorts
│   └── 06_Custom_Sort_Comparators.md   ← sort by lambda, IComparer
│
├── K.Trees/
│   ├── 01_Binary_Tree_Basics.md        ← nodes, height, BFS, DFS
│   ├── 02_DFS_on_Trees.md              ← preorder, inorder, postorder
│   ├── 03_BFS_on_Trees.md              ← level order, zigzag
│   ├── 04_BST_Operations.md            ← insert, delete, validate, O(h)
│   ├── 05_BST_Inorder_is_Sorted.md     ← kth smallest, range queries
│   ├── 06_LCA.md                       ← lowest common ancestor patterns
│   ├── 07_Path_Sum_Problems.md         ← root-to-leaf, any path
│   ├── 08_Serialize_Deserialize.md
│   ├── 09_AVL_Tree.md                  ← rotation, balance factor
│   ├── 10_Trie.md                      ← insert, search, startsWith, word dict
│   ├── 11_Segment_Tree.md              ← range query, point update
│   ├── 12_Fenwick_Tree.md              ← BIT, prefix sum updates
│   └── 13_Tree_Interview_Patterns.md
│
├── L.Heaps_and_Priority_Queue/
│   ├── 01_Heap_Basics.md               ← min/max heap, O(log n) insert
│   ├── 02_Top_K_Elements.md            ← k largest, k smallest
│   ├── 03_Kth_Largest_Element.md       ← quickselect vs heap
│   ├── 04_Merge_K_Sorted_Lists.md
│   ├── 05_Median_of_Stream.md          ← two heaps pattern
│   ├── 06_Task_Scheduling.md           ← frequency map + heap
│   └── 07_Heap_Interview_Patterns.md
│
├── M.Graphs/
│   ├── 01_Graph_Basics.md              ← directed/undirected, adj list/matrix
│   ├── 02_BFS.md                       ← shortest path unweighted, level order
│   ├── 03_DFS.md                       ← traversal, connected components
│   ├── 04_Cycle_Detection.md           ← directed (color), undirected (parent)
│   ├── 05_Topological_Sort.md          ← Kahn's BFS, DFS-based
│   ├── 06_Bipartite_Check.md           ← 2-coloring with BFS/DFS
│   ├── 07_Union_Find_DSU.md            ← union by rank, path compression
│   ├── 08_Dijkstra.md                  ← weighted shortest, O((V+E)logV)
│   ├── 09_Bellman_Ford.md              ← negative weights, O(VE)
│   ├── 10_Floyd_Warshall.md            ← all pairs shortest path
│   ├── 11_MST_Kruskal.md               ← sort edges, DSU
│   ├── 12_MST_Prim.md                  ← greedy, min heap
│   ├── 13_Bridges_and_AP.md            ← Tarjan, low/disc arrays
│   ├── 14_SCC_Kosaraju.md              ← strongly connected components
│   ├── 15_Multi_Source_BFS.md          ← 0-1 BFS, rotting oranges pattern
│   ├── 16_Graph_DP.md                  ← DP on DAG, number of paths
│   └── 17_Graph_Interview_Patterns.md  ← pattern→problem cheat sheet
│
├── N.Dynamic_Programming/
│   ├── 01_DP_Basics.md                 ← memoization vs tabulation
│   ├── 02_1D_DP.md                     ← climbing stairs, house robber, coins
│   ├── 03_2D_DP.md                     ← grid paths, unique paths
│   ├── 04_Knapsack_01.md               ← 0/1 knapsack pattern
│   ├── 05_Unbounded_Knapsack.md        ← coin change, rod cutting
│   ├── 06_LCS_LIS.md                   ← longest common, longest increasing
│   ├── 07_Palindrome_DP.md             ← longest palindrome, partitioning
│   ├── 08_Matrix_Chain_DP.md           ← interval DP pattern
│   ├── 09_DP_on_Trees.md               ← diameter, max path, rerooting
│   ├── 10_DP_on_Graphs.md              ← shortest path DP on DAG
│   ├── 11_Bitmask_DP.md                ← TSP, subsets with state
│   ├── 12_Digit_DP.md                  ← count numbers with constraints
│   └── 13_DP_Interview_Patterns.md
│
├── O.Greedy/
│   ├── 01_Greedy_Basics.md             ← when greedy works, proof sketch
│   ├── 02_Interval_Scheduling.md       ← activity selection, merge intervals
│   ├── 03_Greedy_on_Arrays.md          ← jump game, gas station
│   ├── 04_Huffman_Coding.md
│   └── 05_Greedy_Interview_Patterns.md
│
└── P.Math_and_Bit_Manipulation/
    ├── 01_Bit_Manipulation.md          ← AND OR XOR shifts, common tricks
    ├── 02_Bit_DP_Subsets.md
    ├── 03_Math_Tricks.md               ← mod, pow, GCD, LCM
    ├── 04_Prime_Sieve.md               ← Sieve of Eratosthenes
    └── 05_Bit_Interview_Patterns.md
```

---

## 12_System_Design/

> After DSA — big picture thinking for senior roles.

```
12_System_Design/
│
├── A.Fundamentals/
│   ├── 01_System_Design_Basics.md
│   ├── 02_How_to_Approach_Design_Questions.md
│   ├── 03_Latency_vs_Throughput.md
│   └── 04_Availability_and_Reliability.md
│
├── B.Scalability/
│   ├── 01_Horizontal_Scaling.md
│   ├── 02_Vertical_Scaling.md
│   └── 03_Scalability_Patterns.md
│
├── C.Load_Balancing/
│   ├── 01_What_is_Load_Balancing.md
│   ├── 02_Algorithms.md
│   ├── 03_Health_Checks.md
│   └── 04_Sticky_Sessions.md
│
├── D.Caching/
│   ├── 01_Cache_Basics.md
│   ├── 02_Cache_Aside.md
│   ├── 03_Write_Through.md
│   ├── 04_Write_Back.md
│   ├── 05_Cache_Invalidation.md
│   ├── 06_Cache_Eviction_Policies.md
│   └── 07_Redis_Distributed_Cache.md
│
├── E.Database_Scaling/
│   ├── 01_Replication.md
│   ├── 02_Sharding.md
│   ├── 03_Partitioning.md
│   └── 04_Read_Replicas.md
│
├── F.Distributed_Systems/
│   ├── 01_CAP_Theorem.md
│   ├── 02_Consistent_Hashing.md
│   ├── 03_Fault_Tolerance.md
│   └── 04_Consensus_Algorithms.md
│
├── G.Communication/
│   ├── 01_REST_vs_gRPC.md
│   ├── 02_Message_Queues.md
│   ├── 03_Event_Driven_Architecture.md
│   └── 04_Pub_Sub_Pattern.md
│
├── H.Architecture_Patterns/
│   ├── 01_Monolithic.md
│   ├── 02_Microservices.md
│   ├── 03_API_Gateway.md
│   ├── 04_CQRS.md
│   └── 05_Event_Sourcing.md
│
├── I.API_Design/
│   ├── 01_REST_API_Principles.md
│   ├── 02_HTTP_Methods_and_Status_Codes.md
│   ├── 03_API_Versioning.md
│   ├── 04_Pagination_and_Filtering.md
│   └── 05_GraphQL_Basics.md
│
├── J.Reliability_Patterns/
│   ├── 01_Circuit_Breaker.md
│   ├── 02_Retry_Pattern.md
│   ├── 03_Bulkhead_Pattern.md
│   └── 04_Rate_Limiting.md
│
├── K.Infrastructure/
│   ├── 01_CDN.md
│   ├── 02_Reverse_Proxy.md
│   ├── 03_Auto_Scaling.md
│   └── 04_Docker_Basics.md
│
├── L.Logging_and_Monitoring/
│   ├── 01_Logging_Basics.md
│   ├── 02_Monitoring_and_Alerting.md
│   └── 03_Distributed_Tracing.md
│
└── M.Case_Studies/
    ├── 01_URL_Shortener.md
    ├── 02_Chat_System.md
    ├── 03_Social_Feed.md
    └── 04_Video_Streaming.md
```

---

## 13_Design_Patterns/

> How to write clean, maintainable architecture. Learn after daily frameworks are solid.

```
13_Design_Patterns/
│
├── A.Fundamentals/
│   ├── 01_Design_Pattern_Overview.md
│   ├── 02_Why_Patterns_Matter.md
│   └── 03_SOLID_Quick_Recap.md
│
├── B.Creational_Patterns/
│   ├── 01_Singleton.md
│   ├── 02_Factory_Method.md
│   ├── 03_Abstract_Factory.md
│   ├── 04_Builder.md
│   └── 05_Prototype.md
│
├── C.Structural_Patterns/
│   ├── 01_Adapter.md
│   ├── 02_Bridge.md
│   ├── 03_Composite.md
│   ├── 04_Decorator.md
│   ├── 05_Facade.md
│   ├── 06_Flyweight.md
│   └── 07_Proxy.md
│
├── D.Behavioral_Patterns/
│   ├── 01_Chain_of_Responsibility.md
│   ├── 02_Command.md
│   ├── 03_Iterator.md
│   ├── 04_Mediator.md
│   ├── 05_Observer.md
│   ├── 06_State.md
│   ├── 07_Strategy.md
│   ├── 08_Template_Method.md
│   └── 09_Visitor.md
│
└── E.Patterns_in_ASP_NET/
    ├── 01_Repository_Pattern.md
    ├── 02_Unit_of_Work.md
    ├── 03_Service_Layer_Pattern.md
    └── 04_MVC_Pattern_Deep_Dive.md
```

---

---

## 🗺️ LEARNING ORDER — FINAL

```
╔══════════════════════════════════════════════════════════════╗
║  PHASE 1 — Language First  (Weeks 1–3)                      ║
╠══════════════════════════════════════════════════════════════╣
║  01_CSharp                                                   ║
║    A. Fundamentals → B. Methods → C. OOP (right here)       ║
║    → D. Language Features → E. Collections (deep)           ║
║    → F. LINQ → G-J. rest                                    ║
╚══════════════════════════════════════════════════════════════╝

╔══════════════════════════════════════════════════════════════╗
║  PHASE 2 — Your Daily Stack  (Weeks 4–12)                   ║
╠══════════════════════════════════════════════════════════════╣
║  03_SQL         ← before ADO.NET so you know what to query  ║
║  04_ADO_NET     ← go deep, it's your company standard       ║
║  02_ASP_NET_Core_MVC  ← full framework                      ║
║  06_JavaScript  ← frontend + JSON in same subject           ║
║  07_AJAX        ← daily use                                 ║
║  08_Kendo_UI    ← daily use at work                         ║
╚══════════════════════════════════════════════════════════════╝

╔══════════════════════════════════════════════════════════════╗
║  PHASE 3 — Strengthen  (Weeks 13–18)                        ║
╠══════════════════════════════════════════════════════════════╣
║  05_EF_Core     ← learn after ADO.NET is solid              ║
║  09_Git         ← professional workflow                     ║
║  10_Testing     ← xUnit + Moq, interview must-have          ║
╚══════════════════════════════════════════════════════════════╝

╔══════════════════════════════════════════════════════════════╗
║  PHASE 4 — Senior Level  (Weeks 19–28)                      ║
╠══════════════════════════════════════════════════════════════╣
║  11_DSA         ← quick-recall pattern notes, you know it  ║
║  12_System_Design  ← big picture architecture              ║
║  13_Design_Patterns  ← clean code architecture             ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 📊 FINAL COUNT

| #  | Subject                      | Categories    | Files         |
| -- | ---------------------------- | ------------- | ------------- |
| 01 | CSharp (with OOP)            | 10            | 70            |
| 02 | ASP.NET Core MVC             | 17            | 65            |
| 03 | SQL                          | 10            | 43            |
| 04 | ADO.NET                      | 8             | 36            |
| 05 | EF Core                      | 7             | 25            |
| 06 | JavaScript + JSON            | 6             | 32            |
| 07 | AJAX                         | 5             | 21            |
| 08 | Kendo UI                     | 7             | 34            |
| 09 | Git                          | 5             | 16            |
| 10 | Testing                      | 4             | 15            |
| 11 | **DSA**(algos with DS) | **16**  | **115** |
| 12 | System Design                | 13            | 51            |
| 13 | Design Patterns              | 5             | 27            |
|    | **TOTAL**              | **113** | **550** |

---

## ✅ KEY DECISIONS EXPLAINED

| Decision                                          | Reason                                                                                                                                                             |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **OOP merged into C# at position C**        | You learn language mechanics (vars, methods) then immediately apply them via OOP — not a separate detour                                                          |
| **No standalone OOP subject**               | OOP*is*C# once you know the language. Separate subject = reading the same concepts twice                                                                         |
| **Collections as deep sub-folders**         | Arrays, List, Dictionary, Generics each deserve own folder — they behave differently and are asked separately in interviews                                       |
| **JSON merged into JavaScript/F**           | JSON is a JS data format. Belongs there, not as a 6-chapter standalone subject                                                                                     |
| **DSA algos live WITH their DS**            | Sliding window is an array technique — you learn it while studying arrays. Monotonic stack is a stack technique. No more jumping to a separate Algorithms section |
| **Each DS has Interview_Patterns summary**  | Quick-recall cheat sheet — you've solved the problems, you just need the trigger words                                                                            |
| **DSA → System Design → Design Patterns** | Correct order: solve problems well → think at scale → write clean architecture                                                                                   |
| **Graph folder has 17 files**               | All graph algorithms together — BFS, DFS, Dijkstra, Bellman-Ford, Floyd-Warshall, DSU, Bridges, SCC, Bipartite, MST, DP on graphs — every pattern you need       |

---

*Designed for: **ASP.NET Core MVC + ADO.NET Backend Developer** with strong DSA background*
*Stack: C# · ASP.NET Core · ADO.NET · SQL Server · JavaScript · AJAX · JSON · Kendo UI*
