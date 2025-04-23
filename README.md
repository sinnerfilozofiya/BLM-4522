AdventureWorksLT2022 – Performans Optimizasyonu Demo
====================================================

**Ders Projesi · Veritabanı Performans Optimizasyonu ve İzleme**

Bu README’de raporda kullandığım **tüm T-SQL komutları** sırasıyla yer alır.İsterseniz \*.sql dosyalarına bölebilir, isterseniz aynı sırayla kopyalayıp çalıştırabilirsiniz.

0 · Veritabanını geri yükle
---------------------------

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   sqlCopyEditRESTORE DATABASE AdventureWorksLT2022  FROM DISK = 'C:\Backups\AdventureWorksLT2022.bak'  WITH MOVE 'AdventureWorksLT2012_Data' TO 'C:\MSSQL\Data\AdventureWorksLT2022.mdf',       MOVE 'AdventureWorksLT2012_Log'  TO 'C:\MSSQL\Data\AdventureWorksLT2022.ldf';  GO   `

1 · Query Store’u aç
--------------------

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   sqlCopyEditALTER DATABASE AdventureWorksLT2022  SET QUERY_STORE = ON      (OPERATION_MODE       = READ_WRITE,       QUERY_CAPTURE_MODE   = ALL,       INTERVAL_LENGTH_MINUTES     = 1,       DATA_FLUSH_INTERVAL_SECONDS = 60);  GO   `

2 · Yük oluştur _(Listing 1)_
-----------------------------

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   sqlCopyEditUSE AdventureWorksLT2022;  GO  SET NOCOUNT ON;  DECLARE @i int = 0;  WHILE @i < 100  BEGIN      SELECT SUM(UnitPrice * OrderQty)      FROM SalesLT.SalesOrderDetail      WHERE LineTotal > 1000;      SELECT TOP 1000 *      FROM SalesLT.Product      WHERE Name LIKE '%Bike%';      SET @i += 1;  END  GO  EXEC sys.sp_query_store_flush_db;   -- Query Store’u hemen diske yaz   `

3 · Başlangıç DMV fotoğrafı → **Before.csv**
--------------------------------------------

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   sqlCopyEditSELECT TOP 10         qs.total_worker_time / 1000              AS cpu_ms,         qs.execution_count,         qs.total_elapsed_time / qs.execution_count AS avg_ms,         LEFT(t.text, 100)                        AS query_snippet  FROM sys.dm_exec_query_stats qs  CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) t  ORDER BY cpu_ms DESC;   `

4 · Sorunlu sorgu – **Önce**
----------------------------

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   sqlCopyEditSET STATISTICS IO ON;  SET STATISTICS TIME ON;  GO  SELECT SUM(UnitPrice * OrderQty)  FROM   SalesLT.SalesOrderDetail  WHERE  LineTotal > 1000;  GO  SET STATISTICS IO OFF;  SET STATISTICS TIME OFF;   `

5 · Dizin oluştur _(Listing 2)_
-------------------------------

> İsim çakışıyorsa sonuna 2 ekleyin.

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   sqlCopyEditCREATE NONCLUSTERED INDEX IX_SOD_LineTotal_Cover  ON SalesLT.SalesOrderDetail (LineTotal)  INCLUDE (UnitPrice, OrderQty);  GO   `

6 · Önbellek temizle + sorguyu **sonra** çalıştır
-------------------------------------------------

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   sqlCopyEditDBCC FREEPROCCACHE;  GO  SET STATISTICS IO ON;  SET STATISTICS TIME ON;  GO  SELECT SUM(UnitPrice * OrderQty)  FROM   SalesLT.SalesOrderDetail  WHERE  LineTotal > 1000;  GO  SET STATISTICS IO OFF;  SET STATISTICS TIME OFF;   `

7 · Bakım komutları
-------------------

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   sqlCopyEditUPDATE STATISTICS SalesLT.SalesOrderDetail;  ALTER INDEX ALL ON SalesLT.SalesOrderDetail REBUILD;   -- Express/Std ise ONLINE parametresini kaldırın  GO   `

8 · Profiler izi
----------------

> GUI adımları: Profiler aç → RPC + BatchCompleted seç → Duration ≥ 1000 ms → Run → Stop → SlowQueries.trc olarak kaydet.

9 · RBAC – sadece okuma rolü _(Listing 3)_
------------------------------------------

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   sqlCopyEditUSE AdventureWorksLT2022;  GO  IF NOT EXISTS (SELECT 1 FROM sys.database_principals                 WHERE name = 'PerformanceReader' AND type = 'R')      CREATE ROLE PerformanceReader;  GO  IF NOT EXISTS (SELECT 1 FROM sys.database_principals                 WHERE name = N'BYTE-ME\sinas')      CREATE USER [BYTE-ME\sinas] FOR LOGIN [BYTE-ME\sinas];  GO  EXEC sp_addrolemember 'PerformanceReader', 'BYTE-ME\sinas';  GO   `

10 · Son DMV fotoğrafı → **After.csv**
--------------------------------------

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   sqlCopyEditSELECT TOP 10         qs.total_worker_time / 1000              AS cpu_ms,         qs.execution_count,         qs.total_elapsed_time / qs.execution_count AS avg_ms,         LEFT(t.text, 100)                        AS query_snippet  FROM sys.dm_exec_query_stats qs  CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) t  ORDER BY cpu_ms DESC;   `
