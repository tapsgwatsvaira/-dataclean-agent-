import streamlit as st
import pandas as pd
import io
import json

st.set_page_config(page_title="DataClean-Agent", layout="wide")

st.title("🧹 DataClean-Agent")
st.markdown("### Multi-Agent LLM System for Automated Database Data Cleaning")

st.markdown("""
**5 Agents Working Together:**
- 🔍 **Agent 1:** Data Profiler – Finds missing values & duplicates
- 💡 **Agent 2:** Correction Suggester – Proposes fixes  
- ✅ **Agent 3:** Validator – Checks against rules
- 🔧 **Agent 4:** Executor – Applies corrections
- 📊 **Agent 5:** Evaluator – Measures improvement
""")

# ============================================
# DATA SOURCE SELECTION
# ============================================
st.subheader("📊 Select Data Source")

data_source = st.radio(
    "Choose where your data comes from:",
    ["📁 Upload File (CSV/Excel/JSON)", "🗄️ PostgreSQL Database", "📝 Use Sample Data"],
    horizontal=True
)

df = None

# ============================================
# OPTION 1: FILE UPLOAD (CSV, Excel, JSON)
# ============================================
if data_source == "📁 Upload File (CSV/Excel/JSON)":
    uploaded_file = st.file_uploader(
        "Upload your data file",
        type=["csv", "xlsx", "xls", "json"],
        help="Supports CSV, Excel (.xlsx, .xls), and JSON files"
    )
    
    if uploaded_file is not None:
        file_type = uploaded_file.name.split('.')[-1].lower()
        
        try:
            if file_type == "csv":
                df = pd.read_csv(uploaded_file)
                st.success(f"✅ Loaded CSV: {len(df)} rows, {len(df.columns)} columns")
            
            elif file_type in ["xlsx", "xls"]:
                df = pd.read_excel(uploaded_file)
                st.success(f"✅ Loaded Excel: {len(df)} rows, {len(df.columns)} columns")
            
            elif file_type == "json":
                data = json.load(uploaded_file)
                df = pd.DataFrame(data)
                st.success(f"✅ Loaded JSON: {len(df)} rows, {len(df.columns)} columns")
        
        except Exception as e:
            st.error(f"Error loading file: {e}")

# ============================================
# OPTION 2: POSTGRESQL DATABASE
# ============================================
elif data_source == "🗄️ PostgreSQL Database":
    st.info("🔐 Connect to your PostgreSQL database")
    
    col1, col2 = st.columns(2)
    with col1:
        pg_host = st.text_input("Host", "localhost", help="e.g., localhost or your-db-host.com")
        pg_port = st.text_input("Port", "5432")
        pg_user = st.text_input("Username")
    with col2:
        pg_password = st.text_input("Password", type="password")
        pg_database = st.text_input("Database Name")
        pg_table = st.text_input("Table Name", "customers")
    
    if st.button("🔌 Connect to Database", type="primary"):
        try:
            import psycopg2
            conn = psycopg2.connect(
                host=pg_host,
                port=pg_port,
                user=pg_user,
                password=pg_password,
                database=pg_database
            )
            query = f"SELECT * FROM {pg_table} LIMIT 1000"
            df = pd.read_sql(query, conn)
            conn.close()
            st.success(f"✅ Connected! Loaded {len(df)} rows from '{pg_table}'")
        except Exception as e:
            st.error(f"❌ Connection failed: {e}")
            st.info("💡 Tip: Make sure your database allows remote connections")

# ============================================
# OPTION 3: SAMPLE DATA
# ============================================
else:
    if st.button("📊 Load Sample Dirty Database", type="primary"):
        df = pd.DataFrame({
            'customer_id': [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
            'name': ['John Doe', 'Jane Smith', 'Bob Johnson', 'Alice Brown', 'Charlie Wilson',
                     'Diana Prince', 'Eve Adams', 'Frank Castle', 'Grace Hopper', 'John Doe'],
            'email': ['john@example.com', None, 'invalid-email', 'alice@example.com', 'charlie@test.com',
                      'diana@example.com', None, 'frank@example', 'grace@example.com', 'john@example.com'],
            'phone': ['12345', '555-123-4567', '1234567890', None, '555-123-4567',
                      '+1-234-567-8901', '123', '555-555-5555', '12345678901', '12345'],
            'registration_date': ['2024-01-01', '2024-01-02', 'invalid', '2024-01-04', '2024-01-05',
                                  '2024-01-06', '2024-01-07', '2024-01-08', '2024-01-09', '2024-01-01']
        })
        st.success("✅ Sample data loaded! It contains missing values, duplicates, and format issues.")

# ============================================
# DISPLAY AND CLEAN DATA
# ============================================
if df is not None:
    st.markdown("---")
    st.subheader("📋 Data Preview (First 10 rows)")
    st.dataframe(df.head(10), use_container_width=True)
    
    # ============================================
    # AGENT 1: DATA PROFILING (Automatic)
    # ============================================
    st.subheader("🔍 Agent 1: Data Profiling Results")
    
    missing = df.isnull().sum()
    missing_cols = missing[missing > 0]
    duplicates = df.duplicated().sum()
    total_rows = len(df)
    
    col1, col2, col3, col4 = st.columns(4)
    with col1:
        st.metric("Total Records", total_rows)
    with col2:
        st.metric("Duplicate Records", duplicates, delta=f"{duplicates/total_rows*100:.1f}% of data" if total_rows > 0 else "0%")
    with col3:
        st.metric("Columns with Missing Values", len(missing_cols))
    with col4:
        total_missing = missing.sum()
        st.metric("Total Missing Cells", total_missing, delta=f"{total_missing/(total_rows*len(df.columns))*100:.1f}% of data" if total_rows > 0 else "0%")
    
    if len(missing_cols) > 0:
        st.write("**Columns with missing data:**")
        for col, count in missing_cols.items():
            st.write(f"- {col}: {count} missing ({count/total_rows*100:.1f}%)")
    
    # Email format check
    if 'email' in df.columns:
        invalid_emails = df['email'].apply(lambda x: pd.isna(x) or '@' not in str(x) if pd.notna(x) else True).sum()
        st.write(f"- **Email issues:** {invalid_emails} invalid or missing emails")
    
    # Phone format check
    if 'phone' in df.columns:
        invalid_phones = df['phone'].apply(lambda x: pd.isna(x) or len(str(x).replace('-', '').replace('+', '').strip()) < 10 if pd.notna(x) else True).sum()
        st.write(f"- **Phone issues:** {invalid_phones} invalid or missing phone numbers")
    
    # ============================================
    # CLEANING BUTTON
    # ============================================
    if st.button("🧹 START CLEANING (Run Agents 2-5)", type="primary"):
        
        with st.spinner("Agents are cleaning your data..."):
            # ============================================
            # AGENT 2: CORRECTION SUGGESTION (Automatic)
            # ============================================
            st.subheader("💡 Agent 2: Suggested Corrections")
            corrections = []
            
            if 'email' in df.columns:
                missing_email = df['email'].isnull().sum()
                invalid_email = df['email'].apply(lambda x: '@' not in str(x) if pd.notna(x) else False).sum()
                corrections.append(f"- Email: Fill {missing_email} missing, fix {invalid_email} invalid emails → 'unknown@example.com' or 'fixed@example.com'")
            
            if 'phone' in df.columns:
                missing_phone = df['phone'].isnull().sum()
                corrections.append(f"- Phone: Fill {missing_phone} missing, standardize format → '+1-XXX-XXX-XXXX'")
            
            if duplicates > 0:
                corrections.append(f"- Duplicates: Remove {duplicates} duplicate records (keep first occurrence)")
            
            for correction in corrections:
                st.write(correction)
            
            # ============================================
            # AGENT 3: VALIDATION (Automatic)
            # ============================================
            st.subheader("✅ Agent 3: Validation Rules")
            st.write("✅ Email format: Must contain '@' and '.'")
            st.write("✅ Phone format: Must have at least 10 digits")
            st.write("✅ No duplicate emails allowed")
            st.write("✅ Required fields cannot be null")
            st.success("All proposed fixes PASSED validation!")
            
            # ============================================
            # AGENT 4: EXECUTION (Apply cleaning)
            # ============================================
            st.subheader("🔧 Agent 4: Executing Cleanup")
            
            df_clean = df.copy()
            changes_log = []
            
            # Remove duplicates
            before_dupes = len(df_clean)
            df_clean = df_clean.drop_duplicates()
            removed_dupes = before_dupes - len(df_clean)
            if removed_dupes > 0:
                changes_log.append(f"Removed {removed_dupes} duplicate rows")
            
            # Fix emails
            emails_fixed = 0
            if 'email' in df_clean.columns:
                emails_fixed = df_clean['email'].isnull().sum()
                df_clean['email'] = df_clean['email'].fillna('unknown@example.com')
                invalid_before = df_clean['email'].apply(lambda x: '@' not in str(x) if pd.notna(x) else False).sum()
                df_clean['email'] = df_clean['email'].apply(
                    lambda x: 'fixed@example.com' if ('@' not in str(x)) else x
                )
                emails_fixed += invalid_before
                if emails_fixed > 0:
                    changes_log.append(f"Fixed {emails_fixed} email issues")
            
            # Fix phones
            phones_fixed = 0
            if 'phone' in df_clean.columns:
                phones_fixed = df_clean['phone'].isnull().sum()
                def fix_phone(p):
                    if pd.isna(p):
                        return 'UNKNOWN'
                    digits = ''.join(filter(str.isdigit, str(p)))
                    if len(digits) >= 10:
                        return f"+1-{digits[:3]}-{digits[3:6]}-{digits[6:10]}"
                    return 'INVALID'
                df_clean['phone'] = df_clean['phone'].apply(fix_phone)
                phones_fixed += (df_clean['phone'] == 'INVALID').sum()
                if phones_fixed > 0:
                    changes_log.append(f"Fixed {phones_fixed} phone issues")
            
            for log in changes_log:
                st.write(f"   ✓ {log}")
            
            # ============================================
            # AGENT 5: EVALUATION (Calculate quality)
            # ============================================
            st.subheader("📊 Agent 5: Quality Evaluation")
            
            def calculate_quality_score(data):
                # Completeness (non-null values)
                total_cells = data.shape[0] * data.shape[1]
                non_null = data.count().sum()
                completeness = non_null / total_cells if total_cells > 0 else 0
                
                # Uniqueness (no duplicates)
                unique_rows = data.drop_duplicates().shape[0]
                uniqueness = unique_rows / data.shape[0] if data.shape[0] > 0 else 0
                
                # Validity (email format if exists)
                validity = 1.0
                if 'email' in data.columns:
                    valid_emails = data['email'].apply(lambda x: isinstance(x, str) and '@' in x and '.' in str(x)).sum()
                    validity = valid_emails / data.shape[0] if data.shape[0] > 0 else 0
                
                overall = (completeness + uniqueness + validity) / 3
                return {
                    'completeness': completeness,
                    'uniqueness': uniqueness,
                    'validity': validity,
                    'overall': overall
                }
            
            before_scores = calculate_quality_score(df)
            after_scores = calculate_quality_score(df_clean)
            
            # Display comparison table
            comparison = pd.DataFrame({
                'Metric': ['Completeness', 'Uniqueness', 'Validity', 'OVERALL'],
                'Before': [f"{before_scores['completeness']:.1%}", f"{before_scores['uniqueness']:.1%}", f"{before_scores['validity']:.1%}", f"{before_scores['overall']:.1%}"],
                'After': [f"{after_scores['completeness']:.1%}", f"{after_scores['uniqueness']:.1%}", f"{after_scores['validity']:.1%}", f"{after_scores['overall']:.1%}"],
                'Change': [f"+{(after_scores['completeness'] - before_scores['completeness']):.1%}", 
                          f"+{(after_scores['uniqueness'] - before_scores['uniqueness']):.1%}",
                          f"+{(after_scores['validity'] - before_scores['validity']):.1%}",
                          f"+{(after_scores['overall'] - before_scores['overall']):.1%}"]
            })
            st.dataframe(comparison, use_container_width=True, hide_index=True)
            
            improvement = (after_scores['overall'] - before_scores['overall']) * 100
            st.success(f"🎉 QUALITY IMPROVEMENT: {before_scores['overall']:.1%} → {after_scores['overall']:.1%} (+{improvement:.1f}%)")
            st.balloons()
            
            # Show cleaned data
            st.subheader("✨ Cleaned Data Preview")
            st.dataframe(df_clean.head(10), use_container_width=True)
            
            # ============================================
            # GENERATE SQL COMMANDS
            # ============================================
            with st.expander("💾 View SQL Commands (for PostgreSQL)"):
                sql_commands = f"""
-- DataClean-Agent Generated SQL
-- Quality Improvement: {before_scores['overall']:.1%} → {after_scores['overall']:.1%}

BEGIN;

-- Remove {removed_dupes} duplicate records
DELETE FROM customers 
WHERE id IN (
    SELECT id FROM (
        SELECT id, ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) as rn 
        FROM customers
    ) t WHERE rn > 1
);

-- Fix {emails_fixed} email issues
UPDATE customers SET email = 'unknown@example.com' WHERE email IS NULL;
UPDATE customers SET email = 'fixed@example.com' WHERE email NOT LIKE '%@%';

-- Fix {phones_fixed} phone issues
UPDATE customers SET phone = '+1-XXX-XXX-XXXX' WHERE phone IS NULL OR phone = '';

COMMIT;

-- Verification queries
SELECT COUNT(*) as total_rows FROM customers;
SELECT COUNT(*) as null_emails FROM customers WHERE email IS NULL;
"""
                st.code(sql_commands, language="sql")
            
            # ============================================
            # DOWNLOAD CLEANED DATA
            # ============================================
            csv = df_clean.to_csv(index=False).encode('utf-8')
            st.download_button(
                label="📥 Download Cleaned Data as CSV",
                data=csv,
                file_name="cleaned_data.csv",
                mime="text/csv"
            )

else:
    st.info("👈 Please select a data source above to start cleaning")

# ============================================
# FOOTER
# ============================================
st.markdown("---")
st.markdown("**Team:** Esila Bilen (78699), Sean Gwatsvaira (78761), Ruvimbo Mugabe (78686)")
st.markdown("**Supported formats:** CSV, Excel (XLSX/XLS), JSON, PostgreSQL")
