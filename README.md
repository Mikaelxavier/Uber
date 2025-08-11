
import streamlit as st
import pandas as pd
import sqlite3
import matplotlib.pyplot as plt
from datetime import datetime

# Configuração inicial
st.set_page_config(page_title="Controle Uber", page_icon="icon.png", layout="wide")

# Banco de dados
conn = sqlite3.connect("dados.db")
c = conn.cursor()
c.execute('''CREATE TABLE IF NOT EXISTS registros (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                data TEXT,
                tipo TEXT,
                valor REAL
            )''')
conn.commit()

# Funções
def adicionar_registro(data, tipo, valor):
    c.execute("INSERT INTO registros (data, tipo, valor) VALUES (?, ?, ?)", (data, tipo, valor))
    conn.commit()

def obter_dados():
    return pd.read_sql("SELECT * FROM registros", conn)

# Interface
st.title("📊 Controle Uber - Ganhos e Gastos")

aba = st.sidebar.radio("Menu", ["Adicionar Registro", "Visualizar Dados", "Metas"])

if aba == "Adicionar Registro":
    st.subheader("Adicionar Ganho ou Gasto")
    data = st.date_input("Data", datetime.today())
    tipo = st.selectbox("Tipo", ["Ganho", "Gasto"])
    valor = st.number_input("Valor (R$)", min_value=0.0, step=1.0)
    if st.button("Salvar"):
        adicionar_registro(data.strftime("%Y-%m-%d"), tipo, valor)
        st.success("Registro adicionado com sucesso!")

elif aba == "Visualizar Dados":
    st.subheader("Histórico e Gráficos")
    dados = obter_dados()
    if dados.empty:
        st.info("Nenhum registro encontrado.")
    else:
        st.dataframe(dados)
        ganhos = dados[dados["tipo"] == "Ganho"]["valor"].sum()
        gastos = dados[dados["tipo"] == "Gasto"]["valor"].sum()
        
        fig, ax = plt.subplots()
        ax.bar(["Ganhos", "Gastos"], [ganhos, gastos], color=["green", "red"])
        ax.set_ylabel("R$")
        st.pyplot(fig)

elif aba == "Metas":
    st.subheader("Definir Metas")
    meta_semanal = st.number_input("Meta Semanal (R$)", min_value=0.0, step=10.0)
    meta_mensal = st.number_input("Meta Mensal (R$)", min_value=0.0, step=10.0)
    st.info(f"Meta semanal: R$ {meta_semanal:.2f} | Meta mensal: R$ {meta_mensal:.2f}")

