import psycopg2
import pandas as pd

# ⚠️ Update these with your actual PostgreSQL credentials
DB_HOST = "localhost"
DB_PORT = "5433"   # Default PostgreSQL port (change if needed)
DB_NAME = "question"
DB_USER = "postgres"
DB_PASS = "0452"

def get_db_connection():
    try:
        conn = psycopg2.connect(
            host=DB_HOST,
            port=DB_PORT,
            database=DB_NAME,
            user=DB_USER,
            password=DB_PASS
        )
        return conn
    except Exception as e:
        print(f"❌ Connection failed: {e}")
        return None


# Add transaction
def add_transaction(description, amount, t_type, category):
    conn = get_db_connection()
    if conn is None:
        return "DB connection failed"
    try:
        cur = conn.cursor()
        cur.execute(
            """
            INSERT INTO transactions (description, amount, type, category)
            VALUES (%s, %s, %s, %s)
            """,
            (description, amount, t_type, category)
        )
        conn.commit()
        cur.close()
        conn.close()
        return "✅ Transaction added successfully!"
    except Exception as e:
        return f"❌ Error adding transaction: {e}"


# Get transactions
def get_transactions():
    conn = get_db_connection()
    if conn is None:
        return pd.DataFrame()
    try:
        df = pd.read_sql("SELECT * FROM transactions ORDER BY id ASC", conn)
        conn.close()

        # Rename columns for Streamlit UI
        df.rename(columns={
            "id": "ID",
            "description": "Description",
            "amount": "Amount",
            "type": "Type",
            "category": "Category",
            "created_at": "Created At"
        }, inplace=True)

        return df
    except Exception as e:
        print(f"❌ Error fetching transactions: {e}")
        return pd.DataFrame()


# Update transaction
def update_transaction(t_id, description, amount, t_type, category):
    conn = get_db_connection()
    if conn is None:
        return "DB connection failed"
    try:
        cur = conn.cursor()
        cur.execute(
            """
            UPDATE transactions
            SET description=%s, amount=%s, type=%s, category=%s
            WHERE id=%s
            """,
            (description, amount, t_type, category, t_id)
        )
        conn.commit()
        cur.close()
        conn.close()
        return "✅ Transaction updated successfully!"
    except Exception as e:
        return f"❌ Error updating transaction: {e}"


# Delete transaction
def delete_transaction(t_id):
    conn = get_db_connection()
    if conn is None:
        return "DB connection failed"
    try:
        cur = conn.cursor()
        cur.execute("DELETE FROM transactions WHERE id=%s", (t_id,))
        conn.commit()
        cur.close()
        conn.close()
        return "✅ Transaction deleted successfully!"
    except Exception as e:
        return f"❌ Error deleting transaction: {e}"


# Insights
def get_insights():
    df = get_transactions()
    if df.empty:
        return {"Revenue": 0, "Expense": 0, "Balance": 0}

    revenue = df[df["Type"] == "Revenue"]["Amount"].sum()
    expense = df[df["Type"] == "Expense"]["Amount"].sum()
    balance = revenue - expense
    return {"Revenue": revenue, "Expense": expense, "Balance": balance}
