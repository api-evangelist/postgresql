---
title: plpgsql_wrap v1.0 released
url: https://www.postgresql.org/about/news/plpgsql_wrap-v10-released-3299/
date: '2026-05-20'
author: ''
feed_url: https://postgresql.com/news/rss
---
Toronto, Canada - May 19th, 2026 PostgreSQL plpgsql wrap utility plpgsql_wrap is a PostgreSQL extension used to obfuscate PLPGSQL source code, making it unreadable to users while remaining fully executable by the PostgreSQL database. It is primarily used to protect intellectual property or sensitive business logic when delivering code to third parties. It is similar to the Oracle WRAP function and utility. plpgsql_wrap v1.0 has been released, this is the first public release. It is compatible with PostgreSQL versions >= 12. Write stored procedures with LANGUAGE plpgsql_wrap; source is validated, then AES-256-GCM encrypted directly into pg_proc.prosrc . Users cannot look at store procedures plain text source code any more unless they know the encryption key. pg_dump will not expose store procedures plain text source code too and the pg_dump output is directly restorable. Pre-wrapped blobs are accepted at CREATE time. It is identical to LANGUAGE plpgsql , only the language name differs: plpgsql_wrap . Example of use: CREATE OR REPLACE FUNCTION public.calculate_bonus(emp_id int, yr int)
RETURNS numeric
LANGUAGE plpgsql_wrap
AS $$
DECLARE
    v_salary numeric;
BEGIN
    SELECT salary INTO v_salary FROM employees WHERE id = emp_id;
    RETURN v_salary * 0.15;   -- confidential formula
END;
$$;

-- Calling, permissions, overloading, results - all identical to plpgsql:
SELECT calculate_bonus(42, 2024); In catalog table pg_proc the code is obfuscated: SELECT substring(prosrc, 1, 50)||'...' AS wrapped_code
    FROM pg_proc WHERE proname = 'calculate_bonus';
             wrapped_code                      
-------------------------------------------------------
 PLPGSQLWRAP:1:5752415001f5db365c6c185b66081ff0a23d...
(1 line) This is also what will appears in the pg_dump output. The plpgsql_wrap.unwrap_procedure() function is provided to recover the plain text source code using the encryption key. See detailed information by reading the documentation Links & Credits plpgsql_wrap is an open project. Any contribution to build a better tool is
welcome. You just have to send your ideas, features requests or patches
using the GitHub tools. Links : Documentation: https://github.com/hexacluster/plpgsql_wrap/ Download: https://github.com/hexacluster/plpgsql_wrap/releases/ Community support:  use GitHub report tool at https://github.com/hexacluster/plpgsql_wrap/issues Commercial support: contact Hexacluster About plpgsql_wrap The objective of this extension it to provide a feature to obfuscate plpgsql source code. It is primarily used to protect intellectual property or sensitive business logic when delivering code. This is a project created and maintained by Gilles Darold at HexaCluster Corp .
