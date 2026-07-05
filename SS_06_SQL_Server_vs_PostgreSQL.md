# SQL Server vs PostgreSQL
> 🌐 **labs.postgreshelp.com** | Hands-on Database Engineering Labs

## Learning Objectives

By the end of this chapter, you will be able to:

- Map core SQL Server Linux administration commands to their
  PostgreSQL equivalents
- Recognize that the underlying Linux administration skills transfer
  between database platforms

------------------------------------------------------------------------

  SQL Server                      PostgreSQL
  ------------------------------- -----------------------------
  systemctl status mssql-server   systemctl status postgresql
  1433                            5432
  errorlog                        postgresql.log
  .mdf                            base/`<OID>`{=html}
  .ldf                            pg_wal
  mssql.conf                      postgresql.conf
  sqlcmd                          psql

**Conclusion:** Linux skills transfer across database platforms.

------------------------------------------------------------------------

## Challenge

Without looking back at the table, write down from memory the
PostgreSQL equivalents of `errorlog`, `.ldf`, and `sqlcmd`. Then check
your answers against this chapter.
