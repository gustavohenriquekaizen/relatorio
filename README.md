# relatorio
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

            # Process data for each discipline here (metrics, weak attributes)
            # This part will be implemented in subsequent steps.
            # For now, just indicate the discipline is being processed.
            story.append(Paragraph("... Análise detalhada da disciplina ...", styles["Normal"]))
            story.append(Spacer(1, 6))


    # ============================
    # GERA O PDF
    # ============================
    doc.build(story)
