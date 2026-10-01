# IGN Configurable Detail

An Oracle APEX region plugin that shows records as expandable cards and turns each card into an editable form, configured entirely with SQL.

- **Main query** (region source): one card per row, and each column is a summary cell.
- **Detail query**: the fields shown when a card expands. Each field can be text, number, text area, date picker, select list (LOV) or view only.
- **LOV query**: every option list, loaded once.
- **Dynamic conditions are columns too**: required, shown, read only, cascading lists, values copied from a chosen option, and server-side recalculation.
- **One generic save**: `ign_config_detail.save_edits` writes the changed values back to any table, with an optional lost-update check.

You don't need any page-level JavaScript or PL/SQL beyond the save process.

## Requirements

- Oracle APEX **26.1** or later. The plugin was exported from 26.1 and uses the Procedure plugin interface.
- A schema you can create a package in. This is normally your application's parsing schema.

## Files

| File | What it is |
|---|---|
| `db/ign_config_detail.sql` | PL/SQL package `ign_config_detail`, which renders the plugin, handles its Ajax calls and saves edits. |
| `plugin/region_type_plugin_com_ign_config_detail.sql` | Plugin export `COM.IGN.CONFIG_DETAIL`. The JS and CSS files are already inside it. |
| `src/config-detail.js`, `src/config-detail.css` | Readable copies of the plugin files, for reference only. |
| `sample/01_dv_customers.sql` | Sample table `DV_CUSTOMERS` with 20 fictional customers. |
| `sample/02_page_setup.sql` | The queries and settings for the sample page. Copy and paste them; don't run the file. |
| `docs/REFERENCE.md` | Every attribute, column, suffix and condition, plus the JavaScript API. |

## Install

### Step 1. Install the package

In **SQL Workshop > SQL Scripts** (or SQL Developer / SQLcl, connected as the parsing schema), upload and run `db/ign_config_detail.sql`.

Check that it compiled:

```sql
select object_type, status
  from user_objects
 where object_name = 'IGN_CONFIG_DETAIL';
```

You should get two rows, `PACKAGE` and `PACKAGE BODY`, both `VALID`. If either is `INVALID`, run:

```sql
select line, text from user_errors where name = 'IGN_CONFIG_DETAIL' order by sequence;
```

If the package lives in a different schema from your application's parsing schema, grant execute on it to the parsing schema and create a synonym named `ign_config_detail`.

### Step 2. Import the plugin

1. Open your application, then go to **Shared Components > Plug-ins > Import**.
2. Choose `plugin/region_type_plugin_com_ign_config_detail.sql`, with File Type **Plug-in**.
3. Click **Next**, then **Install Plug-in** into your application.
4. Check that **IGN Configurable Detail** (type Region) appears in the plug-in list.

To install with SQLcl instead:

```sql
begin
    apex_application_install.set_workspace('<YOUR_WORKSPACE>');
    apex_application_install.set_application_id(<YOUR_APP_ID>);
    apex_application_install.generate_offset;
    apex_application_install.set_schema('<YOUR_PARSING_SCHEMA>');
end;
/
@plugin/region_type_plugin_com_ign_config_detail.sql
```

That's the whole installation. The next steps build the sample page.

## Try it with the sample

### Step 3. Create the sample table

Run `sample/01_dv_customers.sql` in the parsing schema. It creates `DV_CUSTOMERS` and inserts 20 rows. A few of the rows are incomplete on purpose, so the dynamic conditions have something to show.

### Step 4. Create the page

1. Create a **Blank Page**, for example page 10, called "Customers".
2. Add a page item **P10_EDITS**: Type **Hidden**, with **Value Protected** set to **No**.
3. Add a region:
   - **Type:** IGN Configurable Detail
   - **Static ID:** `cust`
   - **Source > SQL Query:** the *MAIN QUERY* from `sample/02_page_setup.sql`
4. Open the region's **Attributes** and set:
   - **Primary Key Column:** `CUSTOMER_ID`
   - **Detail Query:** the detail query from the sample file
   - **LOV Query:** the LOV query from the sample file
   - **Edits Item:** `P10_EDITS`
   - **Record Name:** `customer`
   - **Search Placeholder:** `Search customers`
5. Add a button **SAVE** with **Action** set to *Defined by Dynamic Action*. Add a Click dynamic action whose true action is **Execute JavaScript Code**:

   ```js
   apex.region("cust").submit({ request: "SAVE", validate: true });
   ```

6. Under **Processing**, add a PL/SQL process with the save block from the sample file, and set **Server-side Condition** to *Request = Value*, value `SAVE`.
7. Run the page.

### Step 5. What to check

- You see 20 customer cards with search and a count. Clicking a card expands its form.
- Name and Email are required. Clear one and press Save, and the card opens with a message naming the field.
- Typing an address makes City required. Clearing State makes Zip Code read only. These rules come from the `__REQUIRED_WHEN` and `__READONLY_WHEN` columns of the detail query.
- State is a select list filled by the LOV query.
- Change a few cards and press Save. Only the changed fields are sent, the success message says how many customers were saved, and the table shows the new values.
- To see the lost-update check, open the page in two tabs and save the same customer in both. The second save is refused with ORA-20002.

## Use it on your own data

Write your own main query (one row per record) and detail query (filtered with `where <pk> = :PK`). Then point **Primary Key Column** and the save process at your table. `docs/REFERENCE.md` lists everything the queries can return:

- field types
- summary cell decorations (`__SUB`, `__BADGE`, `__PILL`, and so on)
- condition syntax
- `REFRESH_ON` server recalculation
- the JavaScript API (`apex.region("<static id>")`)

## Uninstall

1. Remove the regions that use the plugin.
2. Delete the plugin in **Shared Components > Plug-ins**.
3. Run `drop package ign_config_detail;`.
4. Drop the sample with `drop table dv_customers purge;`.

## Support

If this plugin saves you time, donations are welcome:

**ETH on the Ethereum network (ERC20) only**

```
0xc58187E1b7CE870279CadCB44BF896dCa1f84093
```

<img src="eth-donation-qr.png" alt="QR code for the ETH donation address" width="200">

Please don't send other networks (Base, Arbitrum, BSC, Polygon) or other tokens to this address; they may be lost.

## License

MIT, see `LICENSE`.
