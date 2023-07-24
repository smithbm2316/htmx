+++
title = "Progressively Enhanced Shopping List"
template = "demo.html"
+++


{{ demoenv() }}

<script>
  //=========================================================================
  // Fake Server Side Code
  //=========================================================================

  // data
  var shoppingList = [
    { id: 0, name: "Romaine lettuce", purchased: false },
    { id: 1, name: "Taco seasoning", purchased: false },
    { id: 2, name: "Ground beef", purchased: false },
    { id: 3, name: "Tomatoes", purchased: false },
    { id: 4, name: "Black beans", purchased: false },
    { id: 5, name: "Cheddar cheese", purchased: false },
    { id: 6, name: "Salsa", purchased: false },
    { id: 7, name: "Tortillas", purchased: false },
  ];

  var lastId = 7;

  // routes
  init("/shopping-list", function(req) {
    return displayTemplate(shoppingList);
  });

  onPost("/shopping-list/add", function(req, params) {
    var newItem = {};
    lastId++;
    newItem["id"] = lastId;
    newItem["name"] = params["name"];
    newItem["purchased"] = false;
    shoppingList.push(newItem);

    return shoppingItemTemplate(newItem);
  });

  onPost(/\/shopping-list\/\d+/, function (req, params) {
    var id = parseInt(req.url.split("/")[2]);
    var purchased = params["purchased"];

    for (var i = 0; i < shoppingList.length; i++) {
      if (shoppingList[i]["id"] === id) {
        if (purchased === "on") {
          shoppingList[i]["purchased"] = true;
        } else {
          shoppingList[i]["purchased"] = false;
        }
        return shoppingItemTemplate(shoppingList[i]);
      }
    }
  });

  onDelete(/\/shopping-list\/\d+/, function (req, params) {
    var id = parseInt(req.url.split("/")[2]);

    for (var i = 0; i < shoppingList.length; i++) {
      if (shoppingList[i]["id"] === id) {
        shoppingList.pop(i);
        return "";
      }
    }
  });

  // templates
  function shoppingListTemplate(shoppingList) {
    var shoppingListTemplate = "";
    for (var i = 0; i < shoppingList.length; i++) {
      shoppingListTemplate += shoppingItemTemplate(shoppingList[i]) + "\n";
    }
    return `<tbody>
${shoppingListTemplate}
</tbody>`;
  }

  function shoppingItemTemplate(shoppingItem) {
    var checked = "";
    if (shoppingItem.purchased) {
      checked = "checked";
    }

    return `
<tr hx-target="this" hx-swap="outerHTML">
  <td><input type="checkbox" name="purchased" ${checked} hx-post="/shopping-list/${shoppingItem['id']}" /></td>
  <td>${shoppingItem.name}</td>
  <td><button class="btn primary" type="submit" hx-delete="/shopping-list/${shoppingItem['id']}">Delete</button></td>
</tr>`;
  }

  function formTemplate() {
    return `<form action="/shopping-list/add" method="POST" hx-target="tbody" hx-swap="beforeend">
  <label>
    <span>New Shopping Item</span>
    <input type="text" name="name" />
  </label>
  </div>
  <button class="btn primary" type="submit" hx-post="/shopping-list/add">Add Shopping Item</button>
</form>`;
  }

  function displayTemplate(shoppingList) {
    return `<h2>Shopping List</h2>
${formTemplate()}
  <table>
    <thead>
      <tr>
        <th>Purchased</th>
        <th>Item</th>
        <th></th>
      </tr>
    </thead>
    ${shoppingListTemplate(shoppingList)}
  </table>`;
  }
</script>
