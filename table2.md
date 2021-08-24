// Tables
// [BETA API] API may evolve!
// - Full-featured replacement for old Columns API.
// - See Demo->Tables for details.
// - See ImGuiTableFlags_ and ImGuiTableColumnFlags_ enums for a description of available flags.
// The typical call flow is:
// - 1. Call BeginTable()
// - 2. Optionally call TableSetupColumn() to submit column name/flags/defaults
// - 3. Optionally call TableSetupScrollFreeze() to request scroll freezing of columns/rows
// - 4. Optionally call TableHeadersRow() to submit a header row (names will be pulled from data submitted to TableSetupColumns)
// - 5. Populate contents
//    - In most situations you can use TableNextRow() + TableSetColumnIndex(N) to start appending into a column.
//    - If you are using tables as a sort of grid, where every columns is holding the same type of contents,
//      you may prefer using TableNextColumn() instead of TableNextRow() + TableSetColumnIndex().
//      TableNextColumn() will automatically wrap-around into the next row if needed.
//    - IMPORTANT: Comparatively to the old Columns() API, we need to call TableNextColumn() for the first column!
//    - Both TableSetColumnIndex() and TableNextColumn() return true when the column is visible or performing
//      width measurements. Otherwise, you may skip submitting the contents of a cell/column, BUT ONLY if you know
//      it is not going to contribute to row height.
//      In many situations, you may skip submitting contents for every columns but one (e.g. the first one).
//    - Summary of possible call flow:
//      ----------------------------------------------------------------------------------------------------------
//       TableNextRow() -> TableSetColumnIndex(0) -> Text("Hello 0") -> TableSetColumnIndex(1) -> Text("Hello 1")  // OK
//       TableNextRow() -> TableNextColumn()      -> Text("Hello 0") -> TableNextColumn()      -> Text("Hello 1")  // OK
//                         TableNextColumn()      -> Text("Hello 0") -> TableNextColumn()      -> Text("Hello 1")  // OK: TableNextColumn() automatically gets to next row!
//       TableNextRow()                           -> Text("Hello 0")                                               // Not OK! Missing TableSetColumnIndex() or TableNextColumn()! Text will not appear!
//      ----------------------------------------------------------------------------------------------------------
// - 5. Call EndTable()

bool  BeginTable(const char* str_id, int columns_count, ImGuiTableFlags flags = 0, const ImVec2& outer_size = ImVec2(0, 0), float inner_width = 0.0f);
void  EndTable();                                 // only call EndTable() if BeginTable() returns true!
void  TableNextRow(ImGuiTableRowFlags row_flags = 0, float min_row_height = 0.0f); // append into the first cell of a new row.
bool  TableNextColumn();                          // append into the next column (or first column of next row if currently in last column). Return true when column is visible.
bool  TableSetColumnIndex(int column_n);          // append into the specified column. Return true when column is visible.
int   TableGetColumnIndex();                      // return current column index.
int   TableGetRowIndex();                         // return current row index.

// Tables: Headers & Columns declaration
// - Use TableSetupColumn() to specify label, resizing policy, default width/weight, id, various other flags etc.
//   Important: this will not display anything! The name passed to TableSetupColumn() is used by TableHeadersRow() and context-menus.
// - Use TableHeadersRow() to create a row and automatically submit a TableHeader() for each column.
//   Headers are required to perform: reordering, sorting, and opening the context menu (but context menu can also be available in columns body using ImGuiTableFlags_ContextMenuInBody).
// - You may manually submit headers using TableNextRow() + TableHeader() calls, but this is only useful in some advanced cases (e.g. adding custom widgets in header row).
// - Use TableSetupScrollFreeze() to lock columns (from the right) or rows (from the top) so they stay visible when scrolled.
void  TableSetupColumn(const char* label, ImGuiTableColumnFlags flags = 0, float init_width_or_weight = -1.0f, ImU32 user_id = 0);
void  TableSetupScrollFreeze(int cols, int rows); // lock columns/rows so they stay visible when scrolled.
void  TableHeadersRow();                          // submit all headers cells based on data provided to TableSetupColumn() + submit context menu
void  TableHeader(const char* label);             // submit one header cell manually (rarely used)

// Tables: Miscellaneous functions
// - Most functions taking 'int column_n' treat the default value of -1 as the same as passing the current column index
// - Sorting: call TableGetSortSpecs() to retrieve latest sort specs for the table. Return value will be NULL if no sorting.
//   When 'SpecsDirty == true' you should sort your data. It will be true when sorting specs have changed since last call, or the first time.
//   Make sure to set 'SpecsDirty = false' after sorting, else you may wastefully sort your data every frame!
//   Lifetime: don't hold on this pointer over multiple frames or past any subsequent call to BeginTable().
int                   TableGetColumnCount();                      // return number of columns (value passed to BeginTable)
const char*           TableGetColumnName(int column_n = -1);      // return "" if column didn't have a name declared by TableSetupColumn(). Pass -1 to use current column.
ImGuiTableColumnFlags TableGetColumnFlags(int column_n = -1);     // return column flags so you can query their Enabled/Visible/Sorted/Hovered status flags.
ImGuiTableSortSpecs*  TableGetSortSpecs();                        // get latest sort specs for the table (NULL if not sorting).
void                  TableSetBgColor(ImGuiTableBgTarget bg_target, ImU32 color, int column_n = -1);  // change the color of a cell, row, or column. See ImGuiTableBgTarget_ flags for details.
// Basic use of tables using TableNextRow() to create a new row, and TableSetColumnIndex() to select the column.
// In many situations, this is the most flexible and easy to use pattern.
HelpMarker("Using TableNextRow() + calling TableSetColumnIndex() _before_ each cell, in a loop.");
if (ImGui::BeginTable("##table1", 3))
{
    for (int row = 0; row < 4; row++)
    {
        ImGui::TableNextRow();
        for (int column = 0; column < 3; column++)
        {
            ImGui::TableSetColumnIndex(column);
            ImGui::Text("Row %d Column %d", row, column);
        }
    }
    ImGui::EndTable();
}

// This essentially the same as above, except instead of using a for loop we call TableSetColumnIndex() manually.
// Sometimes this makes more sense.
HelpMarker("Using TableNextRow() + calling TableNextColumn() _before_ each cell, manually.");
if (ImGui::BeginTable("##table2", 3))
{
    for (int row = 0; row < 4; row++)
    {
        ImGui::TableNextRow();
        ImGui::TableNextColumn();
        ImGui::Text("Row %d", row);
        ImGui::TableNextColumn();
        ImGui::Text("Some contents");
        ImGui::TableNextColumn();
        ImGui::Text("123.456");
    }
    ImGui::EndTable();
}

// Another subtle variant, we call TableNextColumn() _before_ each cell. At the end of a row, TableNextColumn() will create a new row.
// Note that we never TableNextRow() here!
HelpMarker(
    "Only using TableNextColumn(), which tends to be convenient for tables where every cells contains the same type of contents.\n"
    "This is also more similar to the old NextColumn() function of the Columns API, and provided to facilitate the Columns->Tables API transition.");
if (ImGui::BeginTable("##table4", 3))
{
    for (int item = 0; item < 14; item++)
    {
        ImGui::TableNextColumn();
        ImGui::Text("Item %d", item);
    }
    ImGui::EndTable();
}