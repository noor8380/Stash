# Working with template files

Stash can read a file, optionally parse it and then cache the output to the database for fast retrieval elsewhere. This can have significant performance advantages over using embeds and snippets. However, it's not meant to be a replacement for EE's core functionality, rather a complement to the existing methods. In particular, Stash templates are best used for query-intensive parts of your templates that are repeated throughout your site such as navigation, footers, etc.

## Config
Create a folder to contain your Stash template files. Ideally this should be above the public webroot of your website.

Open your webroot index.php file, find the "CUSTOM CONFIG VALUES" section and add the following line:

	$assign_to_config['global_vars']['stash_file_basepath'] => '/path/to/stash_templates/'

(of course if you're using a custom config bootstrap file, add the path there instead)
 
## Creating files
Inside your Stash directory, create html files containing template tags, variables etc, just like you would a normal ExpressionEngine template. Name the file after the variable you will be creating and make sure it has the suffix '.html' 

If you want to use contexts to namespace your variables, then create a subfolder named after the context, e.g.

stash_templates/
  my_var1.html
  my_var2.html
  my_context/
     my_var1.html
     my_var2.html

If you are using MSM and want to have separate Stash templates for each site, then you can use each site name as a context. E.g.:

stash_templates/
  my_site1/
     my_var1.html
     my_var2.html
  my_site2/
     my_var1.html
     my_var2.html


## Loading a Stash template

	{exp:stash:get 
		name="main_nav" 			{!-- the name of the variable and the filename if file_name parameter is not set --}
		context="site1" 			{!-- the variable namespace --}
		file_name="my_file" 		{!-- the file name (without the suffix) if different from the variable name, e.g. 'my_file' or 'my_context:my_file'  --}
		scope="site" 				{!-- the scope of the variable, either 'user' or 'site' (default=user) --}
		file="yes" 					{!-- set to yes to tell Stash to look for a file in the Stash template folder (default=no) --}
		save="yes" 					{!-- save the file to the database (default=no)
		refresh="60" 				{!-- how long to cache the template output for (default=1440) --}
		replace="no" 				{!-- do you want the cache to be recreated if it already exists? Set 'yes' while developing, 'no' for production (default=yes)--}
		parse_tags="yes"  			{!-- parse the template tags inside the file? (default=no) --}
		parse_vars="yes"  			{!-- parse variables inside the template? (default=yes, if parse_tags=yes) --}
		parse_conditionals="yes" 	{!-- parse conditionals inside the file? (default=no) --}
		parse_depth = "1" 			{!-- how many passes of the template to make by the parser? 
										 Default is 1, meaning nested tags remain un-parsed. Set to a higher nunber to make repeated passes --}
		output = "yes"				{!-- output the template or just get it for later? (default=yes) --}
	}
	
## Preventing parts of the Stash template from being parsed

When using parse_tags="yes", wrap {stash:nocache}...{/stash:nocache} around content that you do not wish to be parsed.

Here's an example Stash template where we want to parse the Structure Entries tag (which generates a huge number of queries) but still ensure the top level navigation responds to highlight the current page :
	
	{exp:structure_entries depth="3"}
	{if {depth} == 1}{!-- Top Level --}
		<li{stash:nocache}{if "{exp:stash:get name='segment_1' type='global'}" == "{/stash:nocache}{page_url}{stash:nocache}"} class="parent-current"{/if}{/stash:nocache}>
	        <a href="{page_uri}">{title}</a>
		{if {children_total} == 0}{!-- No Children - so close markup --}
			</li>
		{/if}
	{if:else}{!-- Children (not top level) --}
		{if {sibling_count} == 1}{!-- First child - so open markup --}
			<ul class="level{depth}">
		{/if}
	  	<li>
			<a href="{page_uri}">{title}</a>
		{close_markup}
	    	{if {total_children} == 0 || {depth} == {restricted_depth}}
				</li>
	    	{/if}
	    	{if {last_sibling} && {sibling_count} == {sibling_total}}
				</ul>
			</li>
	    {/if}
		{/close_markup}
	{/if}
	{/exp:structure_entries}


## Using placeholders

These are useful if you want to inject variables into the template after it has been parsed and cached. A placeholder takes the form {stash:my_variable}. These will be replaced with the stash variable of the same name wherever the template is output.



