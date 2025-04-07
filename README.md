# TestSELECT 
    child.owner AS child_schema,
    child.table_name AS child_table,
    child.constraint_name AS fk_name,
    parent.owner AS parent_schema,
    parent.table_name AS parent_table,
    parent.constraint_name AS pk_name
FROM 
    all_constraints child
JOIN 
    all_constraints parent
    ON child.r_constraint_name = parent.constraint_name 
    AND child.r_owner = parent.owner
WHERE 
    child.constraint_type = 'R'  -- R = Referential (Foreign Key)
    AND parent.table_name = 'YOUR_PARENT_TABLE_NAME'
    AND parent.owner = 'PARENT_SCHEMA_NAME';
