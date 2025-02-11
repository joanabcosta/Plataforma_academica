import sqlite3
import tkinter as tk
from tkinter import messagebox
import tkinter.ttk as ttk

def conectar_bd():
    return sqlite3.connect("plataforma_academica.db")

def criar_tabelas():
    conn = conectar_bd()
    cursor = conn.cursor()
   
    cursor.execute('''CREATE TABLE IF NOT EXISTS disciplinas (
                        id INTEGER PRIMARY KEY AUTOINCREMENT,
                        nome TEXT NOT NULL,
                        codigo TEXT UNIQUE NOT NULL,
                        descricao TEXT,
                        num_testes INTEGER,
                        num_trabalhos INTEGER,
                        equilibrio_ects_carga INTEGER)''')
   
    cursor.execute('''CREATE TABLE IF NOT EXISTS professores (
                        id INTEGER PRIMARY KEY AUTOINCREMENT,
                        nome TEXT NOT NULL,
                        departamento TEXT NOT NULL,
                        comentario TEXT)''')
   
    cursor.execute('''CREATE TABLE IF NOT EXISTS avaliacoes (
                        id INTEGER PRIMARY KEY AUTOINCREMENT,
                        disciplina_id INTEGER,
                        professor_id INTEGER,
                        dificuldade INTEGER,
                        qualidade_ensino INTEGER,
                        relevancia_conteudo INTEGER,
                        comentario TEXT,
                        dificuldade_testes INTEGER,
                        dificuldade_trabalhos INTEGER,
                        media_notas_passadas REAL,
                        FOREIGN KEY (disciplina_id) REFERENCES disciplinas(id),
                        FOREIGN KEY (professor_id) REFERENCES professores(id))''')
   
    try:
        cursor.execute("ALTER TABLE avaliacoes ADD COLUMN media_ano_anterior REAL")
    except sqlite3.OperationalError:
        pass 
   
    conn.commit()
    conn.close()

def consultar_avaliacoes_disciplinas():
    conn = conectar_bd()
    cursor = conn.cursor()
    cursor.execute("SELECT nome, codigo, descricao, num_testes, num_trabalhos, equilibrio_ects_carga FROM disciplinas")
    disciplinas = cursor.fetchall()
    conn.close()
   
    janela_consulta = tk.Toplevel()
    janela_consulta.title("Disciplinas Cadastradas")
   
    tree = ttk.Treeview(janela_consulta, columns=("Nome", "Código", "Descrição", "Nº Testes", "Nº Trabalhos", "Equilíbrio ECTS"), show="headings")
    tree.heading("Nome", text="Nome")
    tree.heading("Código", text="Código")
    tree.heading("Descrição", text="Descrição")
    tree.heading("Nº Testes", text="Nº Testes")
    tree.heading("Nº Trabalhos", text="Nº Trabalhos")
    tree.heading("Equilíbrio ECTS", text="Equilíbrio ECTS")
   
    for disciplina in disciplinas:
        tree.insert("", "end", values=disciplina)
   
    tree.pack(expand=True, fill="both")

def consultar_avaliacoes_professores():
    conn = conectar_bd()
    cursor = conn.cursor()
    cursor.execute("SELECT nome, departamento, comentario FROM professores")
    professores = cursor.fetchall()
    conn.close()
   
    janela_consulta = tk.Toplevel()
    janela_consulta.title("Professores Cadastrados")
   
    tree = ttk.Treeview(janela_consulta, columns=("Nome", "Departamento", "Comentário"), show="headings")
    tree.heading("Nome", text="Nome")
    tree.heading("Departamento", text="Departamento")
    tree.heading("Comentário", text="Comentário")
   
    for professor in professores:
        tree.insert("", "end", values=professor)
   
    tree.pack(expand=True, fill="both")

def adicionar_disciplina(nome, codigo, descricao, num_testes, num_trabalhos, equilibrio_ects_carga):
    conn = conectar_bd()
    cursor = conn.cursor()
    cursor.execute("INSERT INTO disciplinas (nome, codigo, descricao, num_testes, num_trabalhos, equilibrio_ects_carga) VALUES (?, ?, ?, ?, ?, ?)",
                   (nome, codigo, descricao, num_testes, num_trabalhos, equilibrio_ects_carga))
    conn.commit()
    conn.close()
    messagebox.showinfo("Sucesso", "Disciplina adicionada com sucesso!")

def adicionar_professor(nome, departamento, comentario):
    conn = conectar_bd()
    cursor = conn.cursor()
    cursor.execute("INSERT INTO professores (nome, departamento, comentario) VALUES (?, ?, ?)", (nome, departamento, comentario))
    conn.commit()
    conn.close()
    messagebox.showinfo("Sucesso", "Professor adicionado com sucesso!")
    
def apagar_avaliacao_disciplina():
    def remover():
        disciplina_nome = entry_nome.get()
        conn = conectar_bd()
        cursor = conn.cursor()
        cursor.execute("DELETE FROM avaliacoes WHERE disciplina_id IN (SELECT id FROM disciplinas WHERE nome = ?)", (disciplina_nome,))
        conn.commit()
        conn.close()
        messagebox.showinfo("Sucesso", "Avaliação da disciplina apagada!")
        janela_apagar.destroy()
   
    janela_apagar = tk.Toplevel()
    janela_apagar.title("Apagar Avaliação de Disciplina")
   
    tk.Label(janela_apagar, text="Nome da Disciplina").grid(row=0, column=0)
    entry_nome = tk.Entry(janela_apagar)
    entry_nome.grid(row=0, column=1)
   
    tk.Button(janela_apagar, text="Apagar", command=remover).grid(row=1, columnspan=2)

def apagar_avaliacao_professor():
    def remover():
        professor_nome = entry_nome.get()
        conn = conectar_bd()
        cursor = conn.cursor()
        cursor.execute("DELETE FROM avaliacoes WHERE professor_id IN (SELECT id FROM professores WHERE nome = ?)", (professor_nome,))
        conn.commit()
        conn.close()
        messagebox.showinfo("Sucesso", "Avaliação do professor apagada!")
        janela_apagar.destroy()
   
    janela_apagar = tk.Toplevel()
    janela_apagar.title("Apagar Avaliação de Professor")
   
    tk.Label(janela_apagar, text="Nome do Professor").grid(row=0, column=0)
    entry_nome = tk.Entry(janela_apagar)
    entry_nome.grid(row=0, column=1)
   
    tk.Button(janela_apagar, text="Apagar", command=remover).grid(row=1, columnspan=2)

def interface_adicionar_disciplina():
    def salvar_disciplina():
        adicionar_disciplina(entry_nome.get(), entry_codigo.get(), entry_descricao.get(), int(entry_testes.get()), int(entry_trabalhos.get()), int(entry_equilibrio.get()))
        janela_adicionar.destroy()
   
    janela_adicionar = tk.Toplevel()
    janela_adicionar.title("Adicionar Disciplina")
   
    tk.Label(janela_adicionar, text="Nome").grid(row=0, column=0)
    entry_nome = tk.Entry(janela_adicionar)
    entry_nome.grid(row=0, column=1)
   
    tk.Label(janela_adicionar, text="Código").grid(row=1, column=0)
    entry_codigo = tk.Entry(janela_adicionar)
    entry_codigo.grid(row=1, column=1)
   
    tk.Label(janela_adicionar, text="Descrição").grid(row=2, column=0)
    entry_descricao = tk.Entry(janela_adicionar)
    entry_descricao.grid(row=2, column=1)
   
    tk.Label(janela_adicionar, text="Nº de Testes").grid(row=3, column=0)
    entry_testes = tk.Entry(janela_adicionar)
    entry_testes.grid(row=3, column=1)
   
    tk.Label(janela_adicionar, text="Nº de Trabalhos").grid(row=4, column=0)
    entry_trabalhos = tk.Entry(janela_adicionar)
    entry_trabalhos.grid(row=4, column=1)
   
    tk.Label(janela_adicionar, text="Equilíbrio ECTS/Carga").grid(row=5, column=0)
    entry_equilibrio = tk.Entry(janela_adicionar)
    entry_equilibrio.grid(row=5, column=1)
   
    tk.Button(janela_adicionar, text="Salvar", command=salvar_disciplina).grid(row=6, columnspan=2)

def interface_adicionar_professor():
    def salvar_professor():
        adicionar_professor(entry_nome.get(), entry_departamento.get(), entry_comentario.get())
        janela_adicionar.destroy()
   
    janela_adicionar = tk.Toplevel()
    janela_adicionar.title("Adicionar Professor")
   
    tk.Label(janela_adicionar, text="Nome").grid(row=0, column=0)
    entry_nome = tk.Entry(janela_adicionar)
    entry_nome.grid(row=0, column=1)
   
    tk.Label(janela_adicionar, text="Departamento").grid(row=1, column=0)
    entry_departamento = tk.Entry(janela_adicionar)
    entry_departamento.grid(row=1, column=1)
   
    tk.Label(janela_adicionar, text="Comentários").grid(row=2, column=0)
    entry_comentario = tk.Entry(janela_adicionar)
    entry_comentario.grid(row=2, column=1)
   
    tk.Button(janela_adicionar, text="Salvar", command=salvar_professor).grid(row=3, columnspan=2)
    
def consultar_avaliacoes():
    conn = conectar_bd()
    cursor = conn.cursor()
    cursor.execute("SELECT d.nome AS Disciplina, p.nome AS Professor, a.dificuldade, a.qualidade_ensino, a.relevancia_conteudo, a.comentario, a.media_ano_anterior FROM avaliacoes a JOIN disciplinas d ON a.disciplina_id = d.id JOIN professores p ON a.professor_id = p.id")
    avaliacoes = cursor.fetchall()
    conn.close()
   
    janela_consulta = tk.Toplevel()
    janela_consulta.title("Avaliações Cadastradas")
   
    tree = ttk.Treeview(janela_consulta, columns=("Disciplina", "Professor", "Dificuldade", "Qualidade", "Relevância", "Comentário", "Média Ano Anterior"), show="headings")
    tree.heading("Disciplina", text="Disciplina")
    tree.heading("Professor", text="Professor")
    tree.heading("Dificuldade", text="Dificuldade")
    tree.heading("Qualidade", text="Qualidade")
    tree.heading("Relevância", text="Relevância")
    tree.heading("Comentário", text="Comentário")
    tree.heading("Média Ano Anterior", text="Média Ano Anterior")
   
    for avaliacao in avaliacoes:
        tree.insert("", "end", values=avaliacao)
   
    tree.pack(expand=True, fill="both")

def adicionar_avaliacao():
    def salvar_avaliacao():
        disciplina_nome = entry_disciplina.get()
        professor_nome = entry_professor.get()
        dificuldade = int(entry_dificuldade.get())
        qualidade = int(entry_qualidade.get())
        relevancia = int(entry_relevancia.get())
        comentario = entry_comentario.get()
        media_ano_anterior = float(entry_media_ano_anterior.get())
       
        conn = conectar_bd()
        cursor = conn.cursor()
        cursor.execute("SELECT id FROM disciplinas WHERE nome = ?", (disciplina_nome,))
        disciplina_id = cursor.fetchone()
        cursor.execute("SELECT id FROM professores WHERE nome = ?", (professor_nome,))
        professor_id = cursor.fetchone()
       
        if disciplina_id and professor_id:
            cursor.execute("INSERT INTO avaliacoes (disciplina_id, professor_id, dificuldade, qualidade_ensino, relevancia_conteudo, comentario, media_ano_anterior) VALUES (?, ?, ?, ?, ?, ?, ?)",
                           (disciplina_id[0], professor_id[0], dificuldade, qualidade, relevancia, comentario, media_ano_anterior))
            conn.commit()
            messagebox.showinfo("Sucesso", "Avaliação adicionada com sucesso!")
        else:
            messagebox.showerror("Erro", "Disciplina ou Professor não encontrados.")
       
        conn.close()
        janela_adicionar.destroy()
   
    janela_adicionar = tk.Toplevel()
    janela_adicionar.title("Adicionar Avaliação")
   
    tk.Label(janela_adicionar, text="Nome da Disciplina").grid(row=0, column=0)
    entry_disciplina = tk.Entry(janela_adicionar)
    entry_disciplina.grid(row=0, column=1)
   
    tk.Label(janela_adicionar, text="Nome do Professor").grid(row=1, column=0)
    entry_professor = tk.Entry(janela_adicionar)
    entry_professor.grid(row=1, column=1)
   
    tk.Label(janela_adicionar, text="Dificuldade (1-10)").grid(row=2, column=0)
    entry_dificuldade = tk.Entry(janela_adicionar)
    entry_dificuldade.grid(row=2, column=1)
   
    tk.Label(janela_adicionar, text="Qualidade do Ensino (1-10)").grid(row=3, column=0)
    entry_qualidade = tk.Entry(janela_adicionar)
    entry_qualidade.grid(row=3, column=1)
   
    tk.Label(janela_adicionar, text="Relevância do Conteúdo (1-10)").grid(row=4, column=0)
    entry_relevancia = tk.Entry(janela_adicionar)
    entry_relevancia.grid(row=4, column=1)
   
    tk.Label(janela_adicionar, text="Comentário").grid(row=5, column=0)
    entry_comentario = tk.Entry(janela_adicionar)
    entry_comentario.grid(row=5, column=1)
   
    tk.Label(janela_adicionar, text="Média do Ano Anterior").grid(row=6, column=0)
    entry_media_ano_anterior = tk.Entry(janela_adicionar)
    entry_media_ano_anterior.grid(row=6, column=1)
   
    tk.Button(janela_adicionar, text="Salvar", command=salvar_avaliacao).grid(row=7, columnspan=2)

def interface_principal():
    root = tk.Tk()
    root.title("Plataforma Acadêmica")
   
    tk.Button(root, text="Adicionar Disciplina", command=interface_adicionar_disciplina).pack()
    tk.Button(root, text="Adicionar Professor", command=interface_adicionar_professor).pack()
    tk.Button(root, text="Adicionar Avaliação", command=adicionar_avaliacao).pack()
    tk.Button(root, text="Consultar Disciplinas", command=consultar_avaliacoes_disciplinas).pack()
    tk.Button(root, text="Consultar Professores", command=consultar_avaliacoes_professores).pack()
    tk.Button(root, text="Consultar Avaliações", command=consultar_avaliacoes).pack()
    tk.Button(root, text="Apagar Avaliação de Disciplina", command=apagar_avaliacao_disciplina).pack()
    tk.Button(root, text="Apagar Avaliação de Professor", command=apagar_avaliacao_professor).pack()
    tk.Button(root, text="Sair", command=root.quit).pack()
   
    root.mainloop()

criar_tabelas()
interface_principal()
