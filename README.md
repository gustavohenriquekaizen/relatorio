import pandas as pd
import matplotlib.pyplot as plt
import os
from reportlab.lib.pagesizes import A4
from reportlab.lib import colors
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, Image, Table, TableStyle


from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont
# ============================
# CONFIGURAÇÃO INICIAL
# ============================


ARQUIVO = "Base de provas modificada - Result 1.csv"
PASTA_SAIDA = "relatorios_alunos"


if not os.path.exists(PASTA_SAIDA):
    os.makedirs(PASTA_SAIDA)


# Register a font that supports extended Latin characters
# You might need to upload 'LiberationSans-Regular.ttf' to your Colab environment
try:
    pdfmetrics.registerFont(TTFont('LiberationSans', 'LiberationSans-Regular.ttf'))
except:
    print("Font file 'LiberationSans-Regular.ttf' not found. Please upload it to your Colab environment.")
    # Fallback to a basic font if LiberationSans is not available
    font_name = 'Helvetica'
else:
    font_name = 'LiberationSans'
# ============================
# LEITURA DOS DADOS
# ============================


# Specify encoding to handle potential UnicodeDecodeError
df = pd.read_csv(f"./data/{ARQUIVO}", encoding='latin1')
# Ajusta vírgula decimal se necessário
if df["notaGlobal"].dtype == object:
    df["notaGlobal"] = df["notaGlobal"].str.replace(",", ".").astype(float)


# Cria coluna binária de acerto
df["acertou"] = df["respConsiderada"].str.contains("correta", case=False, na=False)
# Ajusta vírgula decimal se necessário
if df["notaGlobal"].dtype == object:
    df["notaGlobal"] = df["notaGlobal"].str.replace(",", ".").astype(float)


# Cria coluna binária de acerto
df["acertou"] = df["respConsiderada"].str.contains("correta", case=False, na=False)
# Ajusta vírgula decimal se necessário
if df["notaGlobal"].dtype == object:
    df["notaGlobal"] = df["notaGlobal"].str.replace(",", ".").astype(float)


# Cria coluna binária de acerto
df["acertou"] = df["respConsiderada"].str.contains("correta", case=False, na=False)
# ============================
# CÁLCULO DE RANKING POR PROVA
# ============================


# Cada linha tem a notaGlobal do aluno e o idProva — então calculamos o percentil
rankings = (
    df.groupby(["idProva", "idAluno"])["notaGlobal"]
    .mean()
    .groupby("idProva")
    .rank(pct=True)
    .reset_index(name="percentil")
)


# Junta de volta no dataframe principal
df = df.merge(rankings, on=["idProva", "idAluno"], how="left")
# ============================
# FUNÇÃO PARA GERAR PDF POR ALUNO
# ============================
def gerar_relatorio_aluno(aluno_df):
    aluno_nome = aluno_df["aluno"].iloc[0]
    curso = aluno_df["cursoAluno"].iloc[0]
    unidade = aluno_df["unidadeOrgAluno"].iloc[0]
    id_aluno = aluno_df["idAluno"].iloc[0]


    # Cria arquivo PDF
    nome_arquivo = os.path.join(PASTA_SAIDA, f"{aluno_nome.replace('/', '-')}.pdf")
    doc = SimpleDocTemplate(nome_arquivo, pagesize=A4)
    styles = getSampleStyleSheet()


    # Set the font for the styles
    for style_name in styles.byName.keys():
        style = styles[style_name]
        style.fontName = font_name
        if font_name == 'LiberationSans':
             style.leading = 14 # You might need to adjust leading with the new font


    # Add print statement to check the font being used
    print(f"Using font: {font_name}")


    story = []


    # ============================
    # CABEÇALHO
    # ============================


    story.append(Paragraph(f"<b>Relatório de Desempenho Individual</b>", styles["Title"]))
    story.append(Spacer(1, 12))
    story.append(Paragraph(f"<b>Aluno:</b> {aluno_nome}", styles["Normal"]))
    story.append(Paragraph(f"<b>Curso:</b> {curso}", styles["Normal"]))
    story.append(Paragraph(f"<b>Unidade:</b> {unidade}", styles["Normal"]))
    story.append(Spacer(1, 12))


    # ============================
    # RESUMO GERAL
    # ============================


    media_geral = aluno_df["notaGlobal"].mean()
    total_provas = aluno_df["tituloProva"].nunique()
    acertos = aluno_df["acertou"].mean() * 100


    story.append(Paragraph(f"<b>Resumo Geral</b>", styles["Heading2"]))
    story.append(Paragraph(f"Média geral das notas: {media_geral:.2f}", styles["Normal"]))
    story.append(Paragraph(f"Total de provas realizadas: {total_provas}", styles["Normal"]))
    story.append(Paragraph(f"Percentual médio de acertos: {acertos:.1f}%", styles["Normal"]))
    story.append(Spacer(1, 12))


    # ============================
    # DETALHAMENTO POR PROVA
    # ============================


    for (prova, id_prova), df_prova in aluno_df.groupby(["tituloProva", "idProva"]):
        story.append(Paragraph(f"<b>{prova}</b>", styles["Heading2"]))
        story.append(Spacer(1, 6))


        nota = df_prova["notaGlobal"].mean()
        percentil = df_prova["percentil"].iloc[0] * 100


        story.append(Paragraph(f"Sua nota média nesta prova: <b>{nota:.2f}</b>", styles["Normal"]))
        story.append(Paragraph(f"Sua posição percentual entre os colegas: <b>{percentil:.1f}º percentil</b>", styles["Normal"]))
        story.append(Spacer(1, 6))


        # ============================
        # ANÁLISE POR DISCIPLINA (NOVA SEÇÃO)
        # ============================
        story.append(Paragraph(f"<b>Análise por Disciplina nesta Prova</b>", styles["Heading3"]))
        story.append(Spacer(1, 6))


        # Iterate through disciplines within each proof
        for disciplina, df_disciplina in df_prova.groupby("discQuestao"):
            story.append(Paragraph(f"<i>Disciplina: {disciplina}</i>", styles["Normal"]))
            story.append(Spacer(1, 3))


            # Calculate metrics for each discipline
            media_disciplina = df_disciplina["notaGlobal"].mean()
            total_questoes_disciplina = len(df_disciplina)
            percentual_acertos_disciplina = df_disciplina["acertou"].mean() * 100


            story.append(Paragraph(f"Média da nota na disciplina: {media_disciplina:.2f}", styles["Normal"]))
            story.append(Paragraph(f"Total de questões na disciplina: {total_questoes_disciplina}", styles["Normal"]))
            story.append(Paragraph(f"Percentual de acertos na disciplina: {percentual_acertos_disciplina:.1f}%", styles["Normal"]))
            story.append(Spacer(1, 6))




    # ============================
    # GERA O PDF
    # ============================
    doc.build(story)
