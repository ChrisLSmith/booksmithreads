---
layout: custom
title: Home
---

 
  <main class="container">
    <h1>Books I've Read</h1>
    <table id="books-table" class="table">
      <thead>
        <tr>
          <th>Book Title</th>
          <th>Author</th>
          <th>Year Read</th>
          <th>Rating</th>
        </tr>
      </thead>
      <tbody id="books-body">
        <tr id="loading-row">
          <td colspan="4" style="text-align: center; padding: 2rem;">
            <div class="spinner-border text-primary" role="status"></div>
            <div style="margin-top: 0.5rem;">Loading books...</div>
          </td>
        </tr>
      </tbody>
    </table>
  </main>

  <script>
    const sheetId = '1fF7P_rtLpgdr6LadFXJ3ZhjvNVw3TCBHnHi0q4XEx9I';
    const logRange = 'Log!A2:G';

    async function fetchSheetData(range) {
      const url = `https://sheets.googleapis.com/v4/spreadsheets/${sheetId}/values/${range}?key=AIzaSyCFcup6gSBuwCFuwNKF-gIkrK2f_6nfD0g`;
      const response = await fetch(url);
      const data = await response.json();
      return data.values || [];
    }

    function buildBookUrl(title) {
      return `book.html?title=${encodeURIComponent(title)}`;
    }

   function renderBooksTable(data) {
     const tbody = document.querySelector("#books-body");

     // Remove loading row
     const loadingRow = document.getElementById("loading-row");
     if (loadingRow) loadingRow.remove();

     // Sort by date read descending
     data.sort((a, b) => {
       const dateA = new Date(a[3] || '1900-01-01');
       const dateB = new Date(b[3] || '1900-01-01');
       return dateB - dateA;
     });

     const today = new Date();

     data.forEach(row => {
       const title = row[0] || "Unknown Title";
       const author = row[1] || "Unknown Author";
       const dateRead = row[3];
       const year = row[5] || "";
       const rating = row[6] || "";

       let isNew = false;
       if (dateRead) {
         const readDate = new Date(dateRead);
         const timeDiff = today - readDate;
         const daysDiff = timeDiff / (1000 * 60 * 60 * 24);
         isNew = daysDiff <= 5 && daysDiff >= 0;
       }

       const tr = document.createElement("tr");
       tr.innerHTML = `
         <td>
           <a href="${buildBookUrl(title)}">${title}</a>
           ${isNew ? '<span style="color:red; font-weight:bold; margin-left:6px;">🆕 New!</span>' : ''}
         </td>
         <td>${author}</td>
         <td>${year}</td>
         <td>${rating}</td>
       `;
       tbody.appendChild(tr);
     });

     // Initialize DataTables after rendering
    $('#books-table').DataTable({
      paging: true,
      searching: true,
      ordering: true,
      order: [], // Don't override your custom JS sort
      language: {
        searchPlaceholder: "Search books...",
        search: ""
      },
      dom: 'ftlip' // Move the length dropdown to the bottom
    });
   }

    fetchSheetData(logRange).then(renderBooksTable);
  </script>

 
  <!-- jQuery (required for DataTables) -->
  <script src="https://code.jquery.com/jquery-3.7.0.min.js"></script>

  <!-- DataTables JS -->
  <script src="https://cdn.datatables.net/1.13.6/js/jquery.dataTables.min.js"></script>
  <script src="https://cdn.datatables.net/1.13.6/js/dataTables.bootstrap5.min.js"></script>

 