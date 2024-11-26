<details>
<summary style="display: flex; justify-content: space-between; align-items: center; background-color: #28a745; color: white; padding: 10px; border-radius: 5px; cursor:pointer;">
  <span>🚀 Define a foreign key constraint</span>
  <span style="margin-left: auto; font-weight: bold;">📅 November 27, 2024</span>
</summary>

```php
$table->foreignId('organization_id');
```
- This creates an organization_id column in the table.
- The column is an unsigned big integer (BIGINT UNSIGNED) by default, which is the same type used for primary keys when using $table->id()


```php
 ->constrained();
```
- This automatically sets up a foreign key constraint for the organization_id column.
- By default, it assumes the foreign key references the id column on the organizations table (based on the column name organization_id)
```php
$table->foreignId('organization_id')->constrained();

//OR

$table->unsignedBigInteger('organization_id');
$table->foreign('organization_id')->references('id')->on('organizations');
```

# Customizing foreign key setup behavior
If the foreign key references a column other than id or a table with a different name, you can explicitly specify it:
- Field in the Current Table (organization_id)
- Referenced Table (companies)
- Referenced Field (company_id)
```php
$table->foreignId('organization_id')->constrained('companies', 'company_id');

// OR

$table->unsignedBigInteger('organization_id');
$table->foreign('organization_id')
      ->references('company_id')
      ->on('companies');

```

# Foreign key Cascading Options
- cascade: Automatically delete or update dependent rows.
- restrict: Prevent deletion or updates if there are dependent rows.
- set null: Set the foreign key column to NULL if the parent is deleted.
```php
$table->foreignId('organization_id')
      ->constrained()
      ->onDelete('cascade')
      ->onUpdate('cascade');

```

</details>
