
# Proyecto de Análisis de Despidos Globales en MySQL

Este proyecto se divide en dos partes principales: **Limpieza de Datos** y **Análisis Exploratorio de Datos (EDA)**. A continuación, se presentan los pasos más importantes y las consultas utilizadas en cada sección.

## Parte 1: Limpieza de Datos

La limpieza de datos se centra en eliminar duplicados, estandarizar la información, manejar valores nulos y preparar la base de datos para el análisis. Aquí se resumen algunas de las consultas clave:

1. **Eliminación de Duplicados**
   ```sql
   WITH duplicate_cte AS (
       SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
           ) AS row_num
       FROM layoffs_staging
   )
   DELETE 
   FROM duplicate_cte
   WHERE row_num > 1;
   ```

2. **Estandarización de Datos**
   - Remover espacios y estandarizar nombres:
     ```sql
     UPDATE layoffs_staging2
     SET company = TRIM(company);
     ```

   - Corregir inconsistencias en el campo `industry`:
     ```sql
     UPDATE layoffs_staging2
     SET industry = 'Crypto'
     WHERE industry IN ('Crypto Currency', 'CryptoCurrency');
     ```

3. **Normalización de la columna `country`**
   ```sql
   UPDATE layoffs_staging2
   SET country = TRIM(TRAILING '.' FROM country);
   ```

4. **Formateo de la columna `date`**
   ```sql
   UPDATE layoffs_staging2
   SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');
   ALTER TABLE layoffs_staging2
   MODIFY COLUMN `date` DATE;
   ```

5. **Manejo de Valores Nulos y Eliminación de Datos Innecesarios**
   ```sql
   DELETE FROM world_layoffs.layoffs_staging2
   WHERE total_laid_off IS NULL
   AND percentage_laid_off IS NULL;
   ```

6. **Eliminación de columnas innecesarias**
   ```sql
   ALTER TABLE layoffs_staging2
   DROP COLUMN row_num;
   ```

## Parte 2: Análisis Exploratorio de Datos (EDA)

Esta sección explora los datos para extraer información relevante sobre los despidos. A continuación, se presentan algunas de las consultas principales utilizadas:

1. **Desglose por Empresa**
   ```sql
   SELECT company, SUM(total_laid_off) AS total_laid_off
   FROM layoffs_staging2
   GROUP BY company
   ORDER BY total_laid_off DESC
   LIMIT 10;
   ```

2. **Total de Despidos por Industria**
   ```sql
   SELECT industry, SUM(total_laid_off) AS total_laid_off
   FROM layoffs_staging2
   GROUP BY industry
   ORDER BY total_laid_off DESC;
   ```

3. **Total de Despidos por País**
   ```sql
   SELECT country, SUM(total_laid_off) AS total_laid_off
   FROM layoffs_staging2
   GROUP BY country
   ORDER BY total_laid_off DESC;
   ```

4. **Análisis Temporal**
   - Despidos por año:
     ```sql
     SELECT YEAR(date), SUM(total_laid_off)
     FROM layoffs_staging2
     GROUP BY YEAR(date)
     ORDER BY YEAR(date) ASC;
     ```

   - Despidos por mes:
     ```sql
     WITH DATE_CTE AS (
         SELECT SUBSTRING(date,1,7) AS dates, SUM(total_laid_off) AS total_laid_off
         FROM layoffs_staging2
         GROUP BY dates
     )
     SELECT dates, SUM(total_laid_off) OVER (ORDER BY dates ASC) AS rolling_total_layoffs
     FROM DATE_CTE;
     ```

5. **Ranking de Empresas con Más Despidos por Año**
   ```sql
   WITH Company_Year AS (
       SELECT company, YEAR(date) AS years, SUM(total_laid_off) AS total_laid_off
       FROM layoffs_staging2
       GROUP BY company, YEAR(date)
   ),
   Company_Year_Rank AS (
       SELECT company, years, total_laid_off, DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
       FROM Company_Year
   )
   SELECT company, years, total_laid_off, ranking
   FROM Company_Year_Rank
   WHERE ranking <= 3
   ORDER BY years ASC, total_laid_off DESC;
   ```

## Cómo Ejecutar el Proyecto

1. Clona este repositorio:
   ```bash
   git clone https://github.com/tu_usuario/despidos-globales-mysql.git
   ```
2. Importa las tablas iniciales y sigue los pasos de limpieza en **Parte 1**.
3. Ejecuta las consultas en **Parte 2** para analizar y obtener insights de los datos.

## Conclusiones y Resultados

A través de este análisis, se obtuvieron insights importantes sobre el impacto de los despidos en diferentes industrias y regiones a lo largo del tiempo. Se recomienda continuar con un análisis más detallado de cada sector para identificar tendencias y posibles causas.

## Contacto

Si tienes alguna pregunta o sugerencia, puedes contactarme en [LinkedIn](https://www.linkedin.com/in/tu-perfil) o a través de GitHub.

---

¡Gracias por explorar este proyecto conmigo!
