# What is a `join`? 
A `join` is a data operation where usually two tables get combined into one. There are essentially three types of `join`, `full join/ outer join`, `left/right outer join` and `inner join`.
   A `full join` is when every entry from the first table is connected with every entry from the second table resulting in all possible combinations.
   <table>
   <tr><th>First </th><th>Second</th><th>Result</th></tr>
   <tr><td>
      
   |Name| ID |
   |--|--|
   |a| 1|
   |b| 2|
   |c| 2|
   |d| 3|
   
   </td><td>
   
   |Name| ID |
   |--|--|
   |e| 1|
   |f| 2|
   
   </td><td>
   
   |First.Name| First.ID |Second.Name| Second.ID |
   |--|--|--|--|
   |a| 1|e| 1|
   |b| 2|e| 1|
   |c| 2|e| 1|
   |d| 3|e| 1|
   |a| 1|f| 2|
   |b| 2|f| 2|
   |c| 2|f| 2|
   |d| 3|f| 2|

   </td></tr> 
   </table>
      
   An `inner join` is when every is when every entry from the first table gets combined with every entry of the second table that matches some condition, usually that the value of a specific field of the first entry is the same as the value of a specific field of the second entry. Entries from the first table for which no matching entry from the second table can be found get deleted.
   <table>
   <tr><th>First </th><th>Second</th><th>Result</th></tr>
   <tr><td>
      
   |Name| ID |
   |--|--|
   |a| 1|
   |b| 2|
   |c| 2|
   |d| 3|
   
   </td><td>
   
   |Name| ID |
   |--|--|
   |e| 1|
   |f| 2|
   
   </td><td>
   
   |First.Name| ID |Second.Name| 
   |--|--|--|
   |a| 1| e|
   |b| 2| f|
   |c| 2| f|

   </td></tr> 
   </table>

   An `left/right join` is the same as an `inner join`, but entries from the first(`left join`) or the second(`right join`) table dont get geleted even if no match is found for them.
   <table>
   <tr><th>First </th><th>Second</th><th>Result</th></tr>
   <tr><td>
      
   |Name| ID |
   |--|--|
   |a| 1|
   |b| 2|
   |c| 2|
   |d| 3|
   
   </td><td>
   
   |Name| ID |
   |--|--|
   |e| 1|
   |f| 2|
   
   </td><td>
   
   |First.Name| ID |Second.Name| 
   |--|--|--|
   |a| 1| e|
   |b| 2| f|
   |c| 2| f|
   |d| e|  |

   </td></tr> 
   </table>
