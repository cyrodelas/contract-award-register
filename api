const ALLOWED_SHEETS = new Set([
  "4544932179300228", // Project Type
  "640707923496836", // Project
  "1777432421158788", // Contract Packages
  "3541014712373124", // Sub Packages
  "4626949948526468", // Applicability
  "2112606300229508"  // Package Hierarchy
]);

function getCellValue(cell) {
  if (!cell) return "";
  if (cell.displayValue !== undefined && cell.displayValue !== null) {
    return cell.displayValue;
  }
  if (cell.value !== undefined && cell.value !== null) {
    return cell.value;
  }
  return "";
}

export default async function handler(req, res) {
  res.setHeader("Content-Type", "application/json; charset=utf-8");
  res.setHeader("Cache-Control", "no-store, max-age=0");

  if (req.method !== "GET") {
    res.setHeader("Allow", "GET");
    return res.status(405).json({ ok: false, error: "Method not allowed." });
  }

  const sheetId = String(req.query?.sheetId || "").trim();

  if (!sheetId || !ALLOWED_SHEETS.has(sheetId)) {
    return res.status(400).json({ ok: false, error: "Invalid or unauthorized Smartsheet sheet ID." });
  }

  const token = process.env.SMARTSHEET_ACCESS_TOKEN;
  if (!token) {
    return res.status(500).json({
      ok: false,
      error: "SMARTSHEET_ACCESS_TOKEN is not configured in the Vercel environment."
    });
  }

  try {
    const smartsheetResponse = await fetch(`https://api.smartsheet.com/2.0/sheets/${sheetId}`, {
      method: "GET",
      headers: {
        Authorization: `Bearer ${token}`,
        Accept: "application/json",
        "smartsheet-integration-source": "APPLICATION,SharePro,ContractAwardRegister"
      },
      cache: "no-store"
    });

    const rawText = await smartsheetResponse.text();
    let sheet;

    try {
      sheet = rawText ? JSON.parse(rawText) : {};
    } catch {
      return res.status(502).json({
        ok: false,
        error: `Smartsheet returned a non-JSON response (HTTP ${smartsheetResponse.status}).`
      });
    }

    if (!smartsheetResponse.ok) {
      return res.status(smartsheetResponse.status).json({
        ok: false,
        error: sheet?.message || sheet?.errorCode || `Smartsheet API request failed with HTTP ${smartsheetResponse.status}.`
      });
    }

    const columns = Array.isArray(sheet.columns) ? sheet.columns : [];
    const rows = Array.isArray(sheet.rows) ? sheet.rows : [];
    const titleByColumnId = new Map(columns.map(col => [String(col.id), col.title]));

    const mappedRows = rows.map(row => {
      const record = {
        __rowId: row.id ?? null,
        __rowNumber: row.rowNumber ?? null
      };

      for (const cell of row.cells || []) {
        const title = titleByColumnId.get(String(cell.columnId));
        if (!title) continue;
        record[title] = getCellValue(cell);
      }

      return record;
    });

    return res.status(200).json({
      ok: true,
      sheetId,
      sheetName: sheet.name || "",
      totalRows: mappedRows.length,
      rows: mappedRows
    });
  } catch (error) {
    console.error("Smartsheet proxy error:", error);
    return res.status(500).json({
      ok: false,
      error: "Unable to connect to Smartsheet API."
    });
  }
}
