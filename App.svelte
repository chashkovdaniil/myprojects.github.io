<script>
	export let data, titles

  let whatSort // Какой тип данных мы сортируем: dates, strings, numbers
  let noSortedRows
  let columns = titles // Какие столбцы показывать по умолчанию
  /* Тип сортировки: 0 - без сортировки
                     1 - по возрастанию
                     2 - по убыванию
   */
  let typeSort = '0'
  // Функция, которая срабатывает при изменении формы выбора столбца
  function select (event) {
    // Выбранный столбец
    let selectedColumn = {
      'name': event.target.querySelector(`[value="${event.target.value}"]`).innerHTML,
      'type': event.target.value
    }
    // Если выбраны все столбцы
    if (selectedColumn.type === 'All') {
      columns = titles // Столбцы те же, что и в заголовках
      document.querySelector('[name="query"]').disabled = true // Отключаем форму поиска
      document.querySelector('[name="sort"]').disabled = true // Отключаем форму сортировки
      return
    }
    columns = [selectedColumn] // Показываем только выбранный столбец
    whatSort = selectedColumn.type
    document.querySelector('[name="query"]').disabled = false // Включаем форму поиска и сортировки
    document.querySelector('[name="sort"]').disabled = false
  }
  // Функция, которая срабатывает при изменении формы сортировки
  function toSort (event) {
    let table = document.getElementById('table')
    let tbody = table.getElementsByTagName('tbody')[0]
    let rows = [].slice.call(tbody.rows)
    // Функции сортировки
    let compare = {
      'numbers': (a, b) => a.cells[0].innerHTML - b.cells[0].innerHTML,
      'strings': (a, b) => a.cells[0].innerHTML > b.cells[0].innerHTML ? 1 : -1,
      'dates': (a, b) => {
          let regexp = /(\d+).(.+).(\d{4})/;
          let date1 = a.cells[0].innerHTML.split(regexp)
          let date2 = b.cells[0].innerHTML.split(regexp)
          let result = +new Date(date1[3], date1[2], date1[1]) - +new Date(date2[3], date2[2], date2[1])
          return result
        }
      }
    // Сохраняем первоначальные данные, пока нет сортировки. Чтобы можно было использовать выбор "без сортировки"
    if (typeSort === '0') {
      noSortedRows = rows.slice()
    }
    // Сохраняем новый тип сортировки
    typeSort = event.target.value
    rows = noSortedRows.slice()

    if (typeSort === '1') {
      // По возрастанию
      rows.sort(compare[whatSort])
    } else if (typeSort === '2') {
      // По убыванию
      rows.sort(compare[whatSort]).reverse()
    }
    // Убираем тело таблицы
    table.removeChild(tbody)
    // Заменяем тело таблицы
    rows.forEach( (current) => {
      tbody.appendChild(current)
    })
    // Вставляем тело
    table.appendChild(tbody)
  }
</script>

<style>
	h1 {
		color: purple;
	}
  .fullwidth {
    width: 100%;
  }
  .panel {
    width: 100%;
    display: flex;
    justify-content: space-around;
  }
</style>

<h1>Hello! Just below is the table with data.<br>Select a column, then sort or search.</h1>

<div class="panel">
  <div class="panel-item">
    <select on:change={select}>
      <option value="All">All columns</option>
      {#each titles as title}
        <option value={title.type}>{title.name}</option>
      {/each}
    </select>
  </div>
  <div class="panel-item">
    <select disabled name="sort" on:change={toSort}>
      <option value="0">Без сортировки</option>
      <option value="1">По возрастанию</option>
      <option value="2">По убыванию</option>
    </select>
  </div>
  <div class="panel-item">
    <input disabled type="text" name="query" placeholder="Enter query...">
  </div>
</div>

<table class="fullwidth" id="table">
  <thead>
    {#each columns as column}
      <th>{column.name}</th>
    {/each}
  </thead>
  <tbody>
    {#each data as user}
      <tr>
        {#each columns as column}
          <td>{user[column.name.toLowerCase()]}</td>
        {/each}
      </tr>
    {/each}
  </tbody>
</table>
