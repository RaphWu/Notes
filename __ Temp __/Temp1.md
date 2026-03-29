請在 #class:'Calin.ScrewFastening.Views.DataAnalysis':130-596 中加入功能：

- 檔案讀取使用CSVHELPER
- 當 #class:'Calin.ScrewFastening.Views.DataAnalysis':130-596 LOAD時，讀取紀錄列表：
  - 檔名為 `AppPaths.GetDataPath($"{[日期]:yyyyMMdd}.csv")`
  - 設計為方法，可由APP輸入日期做為參數，預設使用今天日期
  - 讀取的資料顯示 #field:'Calin.ScrewFastening.Views.DataAnalysis.dgvRecordList':5224-5280 ，顯示除 Records 以外的資料
- 當點選 #field:'Calin.ScrewFastening.Views.DataAnalysis.dgvRecordList':5224-5280 任一筆時：
  - 將該筆紀錄的 Record 資料在檔名為 `AppPaths.GetDataPath($"{[日期]:yyyyMMdd_HHmmdd}.csv")`
  - 將該筆紀錄的 Record LIST顯示在 #field:'Calin.ScrewFastening.Views.DataAnalysis.dgvDataList':5290-5344
  - 將該筆紀錄的 Record LIST顯示在 #field:'Calin.ScrewFastening.Views.DataAnalysis.plotRecords':5423-5463
