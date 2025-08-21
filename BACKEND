import streamlit as st
import pandas as pd
from backend_fin import (
    add_transaction,
    get_transactions,
    update_transaction,
    delete_transaction,
    get_insights
)

st.set_page_config(page_title="💰 Finance Tracker", layout="centered")

st.title("💰 Personal Finance Tracker")

menu = ["Add Transaction", "View Transactions", "Update Transaction", "Delete Transaction", "Insights"]
choice = st.sidebar.selectbox("Menu", menu)


# --- Add Transaction ---
if choice == "Add Transaction":
    st.subheader("➕ Add New Transaction")
    desc = st.text_input("Description")
    amt = st.number_input("Amount", min_value=0.0, format="%.2f")
    t_type = st.selectbox("Type", ["Revenue", "Expense"])
    cat = st.text_input("Category")

    if st.button("Add Transaction"):
        msg = add_transaction(desc, amt, t_type, cat)
        st.success(msg)


# --- View Transactions ---
elif choice == "View Transactions":
    st.subheader("📄 All Transactions")
    df = get_transactions()
    if df.empty:
        st.warning("No transactions found.")
    else:
        st.dataframe(df)


# --- Update Transaction ---
elif choice == "Update Transaction":
    st.subheader("✏️ Update Transaction")
    df = get_transactions()

    if df.empty:
        st.warning("No transactions available to update.")
    else:
        transaction_id_update = st.number_input("Enter Transaction ID to Update", min_value=1, step=1)

        if transaction_id_update in df["ID"].values:
            row_to_update = df[df["ID"] == int(transaction_id_update)].iloc[0]

            new_desc = st.text_input("Description", row_to_update["Description"])
            new_amt = st.number_input("Amount", value=float(row_to_update["Amount"]))
            new_type = st.selectbox("Type", ["Revenue", "Expense"], 
                                    index=0 if row_to_update["Type"] == "Revenue" else 1)
            new_cat = st.text_input("Category", row_to_update["Category"])

            if st.button("Update Transaction"):
                msg = update_transaction(transaction_id_update, new_desc, new_amt, new_type, new_cat)
                st.success(msg)
        else:
            st.info("Enter a valid Transaction ID.")


# --- Delete Transaction ---
elif choice == "Delete Transaction":
    st.subheader("🗑️ Delete Transaction")
    df = get_transactions()

    if df.empty:
        st.warning("No transactions available to delete.")
    else:
        transaction_id_delete = st.number_input("Enter Transaction ID to Delete", min_value=1, step=1)

        if st.button("Delete Transaction"):
            msg = delete_transaction(transaction_id_delete)
            st.success(msg)


# --- Insights ---
elif choice == "Insights":
    st.subheader("📊 Financial Insights")
    insights = get_insights()
    st.metric("Total Revenue", f"₹{insights['Revenue']:.2f}")
    st.metric("Total Expense", f"₹{insights['Expense']:.2f}")
    st.metric("Balance", f"₹{insights['Balance']:.2f}")
