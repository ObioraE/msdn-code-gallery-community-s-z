# Using multiple SQL-Server databases in a solution
## Requires
- Visual Studio 2015
## License
- MIT
## Technologies
- C#
- SQL Server
- Data Access
- Data Platform
## Topics
- Data Access
- Data
- data and storage
## Updated
- 12/17/2016
## Description

<h1>Description</h1>
<p><span style="font-size:small">This code sample demonstrates using multiple SQL-Server databases for application development. The main idea here is to have a database that its sole purpose is for reference tables that are common to many applications. For
 this example, I&rsquo;ve kept things simple, a database that stores state and city information where other applications that stored data on the server can access this information. A common situation might be that we have a list of customers that have a requirement
 to store contact information such as address and communication details which when done correctly we need to have table for the customer main details, another table for address and another table for communications. I avoided this for an example as it would
 make it harder to learn as there are complexities such as if the customers cross multiple businesses we need to add a field to distinguish this while keeping it simple with customers, state and city is easier to follow along.&nbsp;</span></p>
<p>&nbsp;</p>
<p><span style="font-size:small">Let's look at the databases and tables.</span></p>
<p>&nbsp;</p>
<p><span style="font-size:small">Below the database has two tables, one for states and one for cities which is linked to the state table by StateIdentifier.</span></p>
<p>&nbsp;</p>
<p><span style="font-size:small"><img id="165870" src="165870-f1.jpg" alt="" width="272" height="363"></span></p>
<p><span style="font-size:small">Next is a table in another database where we have two fields that match up to the tables above, StateIdentifier and CityIdentifier</span></p>
<p><img id="165871" src="165871-f2.jpg" alt="" width="276" height="233"></p>
<p><span style="font-size:small">So what we have is in customers when viewing in SQL-Server Management Studio.</span></p>
<p><span style="font-size:small"><img id="165872" src="165872-f3.jpg" alt="" width="382" height="172"></span></p>
<p>&nbsp;</p>
<p><span style="font-size:small">Now let's look at (before peeking at the code) a DataGridView with the data.</span></p>
<p><span style="font-size:small"><img id="165873" src="165873-f4.jpg" alt="" width="465" height="316"><br>
</span></p>
<p><span style="font-size:small">Now let's look at how the query was created in SQL-Server Management Studio, you can do this in Visual Studio also but not as easy and to be honest I used Red Gate SQL-Prompt addin which works in Management Studio and Visual
 Studio.</span></p>
<p><span style="font-size:small">First I created a SELECT query to span both databases using joins so that we get back city and state names.</span></p>
<p><span style="font-size:small"></p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>SQL</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">mysql</span>

<div class="preview">
<pre class="js">SELECT&nbsp;&nbsp;C.FirstName&nbsp;,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;C.LastName&nbsp;,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;C.StateIdentifer&nbsp;,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;C.CityIdentifier&nbsp;,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;us.Name&nbsp;AS&nbsp;State&nbsp;,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;uc.CityName&nbsp;AS&nbsp;City&nbsp;,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;C.id&nbsp;
FROM&nbsp;&nbsp;&nbsp;&nbsp;Customers&nbsp;AS&nbsp;C&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEFT&nbsp;OUTER&nbsp;JOIN&nbsp;CityStateReferences.dbo.US_States&nbsp;AS&nbsp;us&nbsp;ON&nbsp;C.StateIdentifer&nbsp;=&nbsp;us.StateIdentifier&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEFT&nbsp;OUTER&nbsp;JOIN&nbsp;CityStateReferences.dbo.US_Cities&nbsp;AS&nbsp;uc&nbsp;ON&nbsp;uc.CityIdentifier&nbsp;=&nbsp;C.CityIdentifier</pre>
</div>
</div>
</div>
<div class="endscriptcode">Results</div>
<div class="endscriptcode"><img id="165874" src="165874-f5.jpg" alt="" width="517" height="173">&nbsp;</div>
<br>
</span>
<p></p>
<p><span style="font-size:small">The code begins in a class with multiple connection strings, one for each database.</span></p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="csharp">ServerName&nbsp;=&nbsp;<span class="cs__string">&quot;KARENS-PC&quot;</span>;&nbsp;
&nbsp;
ConnectionString&nbsp;=&nbsp;$<span class="cs__string">&quot;Data&nbsp;Source={ServerName};Initial&nbsp;Catalog=CustomersForCodeSample;Integrated&nbsp;Security=True&quot;</span>;&nbsp;
ReferenceConnectionString&nbsp;=&nbsp;$<span class="cs__string">&quot;Data&nbsp;Source={ServerName};Initial&nbsp;Catalog=CityStateReferences;Integrated&nbsp;Security=True&quot;</span>;</pre>
</div>
</div>
</div>
<p><span style="font-size:small">Requesting the data in a method in the operations class. I simply copied the SQL statement from Management Studio into the CommandText for the SqlCommand object. One a DataTable has been loaded several fields are marked as hidden
 so they are not displayed in the DataGridView. Please note that the DataGridView has no pre-defined columns so in this case we can't set visible property of the columns because they don't exists exit. I favor this when possible because if the form becomes
 corrupt you can use a new form, place the DataGridView, BindingSource onto that form with the code written and we are back in business.</span></p>
<p></p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="js">public&nbsp;bool&nbsp;GetCustomers()&nbsp;
<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">var</span>&nbsp;success&nbsp;=&nbsp;false;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;CustomersTable&nbsp;=&nbsp;<span class="js__operator">new</span>&nbsp;DataTable();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;using&nbsp;(SqlConnection&nbsp;cn&nbsp;=&nbsp;<span class="js__operator">new</span>&nbsp;SqlConnection&nbsp;<span class="js__brace">{</span>&nbsp;ConnectionString&nbsp;=&nbsp;ConnectionString&nbsp;<span class="js__brace">}</span>)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;using&nbsp;(SqlCommand&nbsp;cmd&nbsp;=&nbsp;<span class="js__operator">new</span>&nbsp;SqlCommand&nbsp;<span class="js__brace">{</span>&nbsp;Connection&nbsp;=&nbsp;cn<span class="js__brace">}</span>)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cmd.CommandText&nbsp;=&nbsp;@&quot;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SELECT&nbsp;&nbsp;C.id,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;C.FirstName&nbsp;,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;C.LastName&nbsp;,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;C.CityIdentifier&nbsp;,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;C.StateIdentifer&nbsp;,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;uc.CityName&nbsp;AS&nbsp;City&nbsp;,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;us.Name&nbsp;AS&nbsp;State&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FROM&nbsp;&nbsp;&nbsp;&nbsp;Customers&nbsp;AS&nbsp;C&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEFT&nbsp;OUTER&nbsp;JOIN&nbsp;CityStateReferences.dbo.US_States&nbsp;AS&nbsp;us&nbsp;ON&nbsp;C.StateIdentifer&nbsp;=&nbsp;us.StateIdentifier&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEFT&nbsp;OUTER&nbsp;JOIN&nbsp;CityStateReferences.dbo.US_Cities&nbsp;AS&nbsp;uc&nbsp;ON&nbsp;uc.CityIdentifier&nbsp;=&nbsp;C.CityIdentifier&quot;;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">try</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cn.Open();&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CustomersTable.Load(cmd.ExecuteReader());&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__sl_comment">//&nbsp;field&nbsp;we&nbsp;want&nbsp;but&nbsp;not&nbsp;to&nbsp;show&nbsp;in&nbsp;the&nbsp;user&nbsp;interface</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CustomersTable.Columns[<span class="js__string">&quot;id&quot;</span>].ColumnMapping&nbsp;=&nbsp;MappingType.Hidden;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CustomersTable.Columns[<span class="js__string">&quot;StateIdentifer&quot;</span>].ColumnMapping&nbsp;=&nbsp;MappingType.Hidden;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CustomersTable.Columns[<span class="js__string">&quot;CityIdentifier&quot;</span>].ColumnMapping&nbsp;=&nbsp;MappingType.Hidden;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__sl_comment">//&nbsp;better&nbsp;to&nbsp;perform&nbsp;sort&nbsp;on&nbsp;client&nbsp;side</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CustomersTable.DefaultView.Sort&nbsp;=&nbsp;<span class="js__string">&quot;LastName&quot;</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;success&nbsp;=&nbsp;true;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">catch</span>&nbsp;(Exception&nbsp;ex)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HasErrors&nbsp;=&nbsp;true;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ExceptionMessage&nbsp;=&nbsp;ex.Message;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">return</span>&nbsp;success;&nbsp;
&nbsp;
<span class="js__brace">}</span>&nbsp;
</pre>
</div>
</div>
</div>
<div class="endscriptcode">&nbsp;</div>
<p></p>
<p><span style="font-size:small">So all is well with displaying existing data, now let's think about adding new data in a simple format. The focus here is on setting the state and city for a new customer.</span></p>
<p><img id="165875" src="165875-f6.jpg" alt="" width="511" height="363"></p>
<p><span style="font-size:small">When examing the code in the Operation class I've setup methods to get cities and states by first writing the queries in Management Studio and testing them to ensure they work so if they fail in the project we know it's not
 server side problems.</span></p>
<p><span style="font-size:small">Example, get cities for Oregon (in the app we can pare down the fields returned)</span></p>
<p><span style="font-size:small"></p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>SQL</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">mysql</span>

<div class="preview">
<pre class="js">SELECT&nbsp;&nbsp;C.CityIdentifier&nbsp;,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;C.CityName&nbsp;,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;us.Name&nbsp;,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;us.Abbreviation&nbsp;
FROM&nbsp;&nbsp;&nbsp;&nbsp;CityStateReferences.dbo.US_Cities&nbsp;AS&nbsp;C&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEFT&nbsp;OUTER&nbsp;JOIN&nbsp;CityStateReferences.dbo.US_States&nbsp;AS&nbsp;us&nbsp;ON&nbsp;C.StateIdentifier&nbsp;=&nbsp;us.StateIdentifier&nbsp;
WHERE&nbsp;&nbsp;&nbsp;(&nbsp;C.StateIdentifier&nbsp;=&nbsp;<span class="js__num">41</span>&nbsp;);</pre>
</div>
</div>
</div>
<div class="endscriptcode">Okay, the most important part to look at in the edit form (shown above) is the SelectionIndexChanged event of the State ComboBox e.g.</div>
<div class="endscriptcode"></div>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="js">private&nbsp;<span class="js__operator">void</span>&nbsp;cboState_SelectedIndexChanged(object&nbsp;sender,&nbsp;EventArgs&nbsp;e)&nbsp;
<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;cboCity.DataSource&nbsp;=&nbsp;null;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;cboCity.DisplayMember&nbsp;=&nbsp;<span class="js__string">&quot;&quot;</span>;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__sl_comment">//&nbsp;fix&nbsp;bug&nbsp;with&nbsp;ComboBox&nbsp;reload</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;cboCity.Items.Add(<span class="js__string">&quot;&quot;</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;cboCity.Items.Clear();&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">var</span>&nbsp;id&nbsp;=&nbsp;((State)cboState.SelectedItem).StateIdentifier;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">var</span>&nbsp;result&nbsp;=&nbsp;cities.Where(city&nbsp;=&gt;&nbsp;city.StateIdentifier&nbsp;==&nbsp;id).ToList();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">if</span>&nbsp;(result.Count&nbsp;&gt;<span class="js__num">0</span>&nbsp;&amp;&amp;&nbsp;result.First().Name&nbsp;!=&nbsp;<span class="js__string">&quot;Select&nbsp;one&quot;</span>)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result.Insert(<span class="js__num">0</span>,&nbsp;<span class="js__operator">new</span>&nbsp;City&nbsp;<span class="js__brace">{</span>&nbsp;CityIdentifier&nbsp;=&nbsp;<span class="js__num">0</span>,&nbsp;Name&nbsp;=&nbsp;<span class="js__string">&quot;Select&nbsp;one&quot;</span>&nbsp;<span class="js__brace">}</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cboCity.DataSource&nbsp;=&nbsp;result;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cboCity.DisplayMember&nbsp;=&nbsp;<span class="js__string">&quot;Name&quot;</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
<span class="js__brace">}</span></pre>
</div>
</div>
</div>
<div class="endscriptcode">There is code in the Operation class that needs to be studied for the above yet the majority is shown that I want you to focus on. We only allow cities for the selected state so we need to use a little Lambda to do this.</div>
</span>
<p></p>
<p><span style="font-size:small">Next thing of importance, since we are not strong typed like in Enity Framework I wrote language exension methods to make life easier so that to get at say a CityIdentifier for the current row in the DataGridView we can write
 bsData.CityIdentifier rather than introduce the casting.</span></p>
<p><span style="font-size:small">To round things out, focus more on the idea of having reference databases for multiple solutions and not so much on the code as this could had been an ASP.NET, WPF etc solution. At my work place we have many applications sharing
 reference databases where none of the applications know or can access the other applications data.</span></p>
<p><strong><span style="font-size:2em; color:#ff0000">More Information</span></strong></p>
<p><span style="font-size:small">Before attempting to run the project you need to run the scipts below. They will create both databases and populate them with data. Please note not all states have cities so if you pick a state with no cities that is by design
 or should I say I didn't want to write out that many cities.</span></p>
<p><span style="font-size:small"><img id="165877" src="165877-setup.jpg" alt="" width="350" height="254"><br>
</span></p>
<p><span style="font-size:small"><br>
</span></p>
