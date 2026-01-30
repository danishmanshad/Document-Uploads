# Document-Uploads
Just for upload documents
import pandas as pd
import matplotlib.pyplot as plt
from matplotlib.gridspec import GridSpec

excel_file = "aero_test_data.xlsx"

compare = {
    "Sheet1": {"runs": [5861], "tag": "New"},
    "Sheet2": {"runs": [5851], "tag": "Old"}
}
# FUNCTION: PARSE ONE SHEET
def parse_sheet(excel_file, sheet_name):

    df = pd.read_excel(excel_file, sheet_name=sheet_name, header=None)

    runs = {}
    meta_data = {}

    i = 0
    nrows = len(df)

    while i < nrows:

        if pd.notna(df.iloc[i, 0]) and isinstance(df.iloc[i, 0], (int, float)):
            run_no = int(df.iloc[i, 0])

            header = df.iloc[i]
            meta = {
                "M": header[3],
                "LEF": header[4],
                "B": header[9]
            }

            var_names = df.iloc[i + 1, 1:].tolist()

            data = []
            j = i + 2
            while j < nrows and pd.notna(df.iloc[j, 1]):
                data.append(df.iloc[j, 1:].tolist())
                j += 1

            run_df = pd.DataFrame(data, columns=var_names)
            run_df = run_df.apply(pd.to_numeric, errors="coerce")

            runs[run_no] = run_df
            meta_data[run_no] = meta

            i = j
        else:
            i += 1

    return runs, meta_data
# MAIN PLOTTING (HORIZONTAL LAYOUT)
fig = plt.figure(figsize=(15, 5))  # wide figure for horizontal layout
gs = GridSpec(1, 3, width_ratios=[1, 1, 1], wspace=0.3)  # 1 row, 3 columns

# Create three axes side by side
ax_cl = fig.add_subplot(gs[0, 0])
ax_cd = fig.add_subplot(gs[0, 1])
ax_cm = fig.add_subplot(gs[0, 2])

title_set = False

for sheet, cfg in compare.items():

    run_db, meta_db = parse_sheet(excel_file, sheet)

    for run in cfg["runs"]:
        if run not in run_db:
            continue

        df_run = run_db[run]

        ax_cl.plot(df_run["alpha"], df_run["CL"],
                   marker='o', linewidth=2,
                   label=f'{cfg["tag"]}_{run}')

        ax_cd.plot(df_run["alpha"], df_run["CD"],
                   marker='o', linewidth=2,
                   label=f'{cfg["tag"]}_{run}')

        ax_cm.plot(df_run["alpha"], df_run["Cm"],
                   marker='o', linewidth=2,
                   label=f'{cfg["tag"]}_{run}')

        if not title_set:
            meta = meta_db[run]
            fig.suptitle(
                f'M {meta["M"]}, LEF {meta["LEF"]}, B {meta["B"]}',
                fontsize=14,
                fontweight='bold'
            )
            title_set = True
# AXIS FORMATTING
ax_cl.set_title(r'$C_L$ vs $\alpha$')
ax_cd.set_title(r'$C_D$ vs $\alpha$')
ax_cm.set_title(r'$C_m$ vs $\alpha$')

ax_cl.set_xlabel(r'$\alpha$ (deg)')
ax_cd.set_xlabel(r'$\alpha$ (deg)')
ax_cm.set_xlabel(r'$\alpha$ (deg)')

ax_cl.set_ylabel(r'$C_L$')
ax_cd.set_ylabel(r'$C_D$')
ax_cm.set_ylabel(r'$C_m$')

for ax in [ax_cl, ax_cd, ax_cm]:
    ax.grid(True, linestyle='--', alpha=0.6)
    ax.legend(frameon=True, fontsize=10)

plt.tight_layout(rect=[0, 0, 1, 0.93])
plt.show()
