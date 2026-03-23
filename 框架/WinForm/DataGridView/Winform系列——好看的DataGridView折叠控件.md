---
aliases:
date: 2015-07-01
update:
author: 懒得安分
language:
sourceurl: https://www.cnblogs.com/landeanfen/p/4600051.html
tags:
  - CSharp
  - WinForm
  - DataGridView
---

# 好看的 DataGridView 折叠控件

来园子几年了，第一次写博客。以前看到别人的博客就在想：这些人怎么能有这么多时间整理这么多知识，难道他们不用工作、不用写代码、不用交付测试？随着工作阅历的增加，发现其实并不是时间的问题，关键一个字：懒。其实写博客的好处大家伙都心知肚明。呵呵，第一次写就这么多废话，看样子真是年纪大了。

其实自己之前的 5 年也一直是做 BS 的系统，现在刚换的一家公司需要做一个 CS 的产品。屌了，自己之前一点经验都没有呢，没办法，既来之则安之，学呗。于是乎各种百度、各种视频，各种资料。系统其中一个需求就是需要表格折叠显示，这如果在 BS 里面那太简单了，JqGrid 默认都自带，可是 DataGridview 不支持折叠啊，咋办。自己封装呗，于是乎又是各种百度，这种源码学习。最后借鉴源码封了这么一个东西，发出来分享下，也能让自己加深印象。首先不多说，上图：

![[好看的DataGridView折叠控件-1.png]]

 ![[好看的DataGridView折叠控件-2.png]]
 ![[好看的DataGridView折叠控件-3.png]]
 ![[好看的DataGridView折叠控件-4.png]]
大概的效果就是这样。上代码。

1. 首先重写 DataGridview，代码如下：

```csharp
public class MasterControl : DataGridView
{
	#region 字段
	private List<int> rowCurrent = new List<int>();
	internal static int rowDefaultHeight = 22;
	internal static int rowExpandedHeight = 300;
	internal static int rowDefaultDivider = 0;
	internal static int rowExpandedDivider = 300 - 22;
	internal static int rowDividerMargin = 5;
	internal static bool collapseRow;
 //detailControl变量作为一个容器用来保存子表格
	public detailControl childView = new detailControl() { Visible = false }; // VBConversions Note: Initial value cannot be assigned here since it is non-static.  Assignment has been moved to the class constructors.
	//
	internal System.Windows.Forms.ImageList RowHeaderIconList;
	private System.ComponentModel.Container components = null;
	//
	DataSet _cDataset;
	string _foreignKey;
	string _primaryKey;
	string _filterFormat;
	private controlType EControlType;
	public int ExpandRowIndex = 0;


	#endregion

	#region 构造函数
	/// <summary>
	/// 通过传递过来的枚举判断是两级还是三级展开，表的对应关系通过Relations来读取
	/// 所以调用此构造函数的时候必须要讲Relations设置正确，才能正确显示层级关系。
	///  oDataSet.Relations.Add("1", oDataSet.Tables["T1"].Columns["Menu_ID"], oDataSet.Tables["T2"].Columns["Menu_ID"]);
	///  oDataSet.Relations.Add("2", oDataSet.Tables["T2"].Columns["Menu_Name2"], oDataSet.Tables["T3"].Columns["Menu_Name2"]);
	///  这两次Add的顺序不能颠倒，必须先添加一、二级的表关联，再添加二、三级的表关联
	/// </summary>
	/// <param name="cDataset">数据源DataSet，里面还有各个表的对应关系</param>
	/// <param name="eControlType">枚举类型</param>
	public MasterControl(DataSet cDataset, controlType eControlType)
	{
		SetMasterControl(cDataset, eControlType);
	}

	/// <summary>
	/// 第二种使用方法
	/// </summary>
	/// <param name="lstData1">折叠控件第一层的集合</param>
	/// <param name="lstData2">折叠控件第二层的集合</param>
	/// <param name="lstData3">折叠控件第三层的集合</param>
	/// <param name="dicRelateKey1">第一二层之间对应主外键</param>
	/// <param name="dicRelateKey2">第二三层之间对应主外键</param>
	/// <param name="eControlType">枚举类型</param>
	public MasterControl(object lstData1, object lstData2,
						 object lstData3, Dictionary<string, string> dicRelateKey1,
						 Dictionary<string ,string>dicRelateKey2, controlType eControlType)
	{
		var oDataSet = new DataSet();
		try
		{
			var oTable1 = new DataTable();
			oTable1 = Fill(lstData1);
			oTable1.TableName = "T1";

			var oTable2 = Fill(lstData2);
			oTable2.TableName = "T2";

			if (lstData3 == null || dicRelateKey2 == null || dicRelateKey2.Keys.Count <= 0)
			{
				oDataSet.Tables.AddRange(new DataTable[] { oTable1, oTable2 });
				oDataSet.Relations.Add("1", oDataSet.Tables["T1"].Columns[dicRelateKey1.Keys.FirstOrDefault()], oDataSet.Tables["T2"].Columns[dicRelateKey1.Values.FirstOrDefault()]);
			}
			else
			{
				var oTable3 = Fill(lstData3);
				oTable3.TableName = "T3";

				oDataSet.Tables.AddRange(new DataTable[] { oTable1, oTable2, oTable3 });
				//这是对应关系的时候主键必须唯一
				oDataSet.Relations.Add("1", oDataSet.Tables["T1"].Columns[dicRelateKey1.Keys.FirstOrDefault()], oDataSet.Tables["T2"].Columns[dicRelateKey1.Values.FirstOrDefault()]);
				oDataSet.Relations.Add("2", oDataSet.Tables["T2"].Columns[dicRelateKey2.Keys.FirstOrDefault()], oDataSet.Tables["T3"].Columns[dicRelateKey2.Values.FirstOrDefault()]);
			}
		}
		catch
		{
			oDataSet = new DataSet();
		}
		SetMasterControl(oDataSet, eControlType);
	}

	/// <summary>
	/// 控件初始化
	/// </summary>
	private void InitializeComponent()
	{
		this.components = new System.ComponentModel.Container();
		base.RowHeaderMouseClick += new System.Windows.Forms.DataGridViewCellMouseEventHandler(MasterControl_RowHeaderMouseClick);
		base.RowPostPaint += new System.Windows.Forms.DataGridViewRowPostPaintEventHandler(MasterControl_RowPostPaint);
		base.Scroll += new System.Windows.Forms.ScrollEventHandler(MasterControl_Scroll);
		base.SelectionChanged += new System.EventHandler(MasterControl_SelectionChanged);
		System.ComponentModel.ComponentResourceManager resources = new System.ComponentModel.ComponentResourceManager(typeof(MasterControl));
		this.RowHeaderIconList = new System.Windows.Forms.ImageList(this.components);
		((System.ComponentModel.ISupportInitialize)this).BeginInit();
		this.SuspendLayout();
		//
		//RowHeaderIconList
		//
		this.RowHeaderIconList.ImageStream = (System.Windows.Forms.ImageListStreamer)(resources.GetObject("RowHeaderIconList.ImageStream"));
		this.RowHeaderIconList.TransparentColor = System.Drawing.Color.Transparent;
		this.RowHeaderIconList.Images.SetKeyName(0, "expand.png");
		this.RowHeaderIconList.Images.SetKeyName(1, "collapse.png");
		//
		//MasterControl
		//
		((System.ComponentModel.ISupportInitialize)this).EndInit();
		this.ResumeLayout(false);

	}
	#endregion

	#region 数据绑定
	/// <summary>
	/// 设置表之间的主外键关联
	/// </summary>
	/// <param name="tableName">DataTable的表名称</param>
	/// <param name="foreignKey">外键</param>
	public void setParentSource(string tableName, string primarykey, string foreignKey)
	{
		this.DataSource = new DataView(_cDataset.Tables[tableName]);
		cModule.setGridRowHeader(this);
		_foreignKey = foreignKey;
		_primaryKey = primarykey;
		if (_cDataset.Tables[tableName].Columns[primarykey].GetType().ToString() == typeof(int).ToString()
			|| _cDataset.Tables[tableName].Columns[primarykey].GetType().ToString() == typeof(double).ToString()
			|| _cDataset.Tables[tableName].Columns[primarykey].GetType().ToString() == typeof(decimal).ToString())
		{
			_filterFormat = foreignKey + "={0}";
		}
		else
		{
			_filterFormat = foreignKey + "=\'{0}\'";
		}
	}
	#endregion

	#region 事件
	//控件的行头点击事件
	private void MasterControl_RowHeaderMouseClick(object sender, DataGridViewCellMouseEventArgs e)
	{
		try
		{
			Rectangle rect = new Rectangle(System.Convert.ToInt32((double)(rowDefaultHeight - 16) / 2), System.Convert.ToInt32((double)(rowDefaultHeight - 16) / 2), 16, 16);
			if (rect.Contains(e.Location))
			{
				//缩起
				if (rowCurrent.Contains(e.RowIndex))
				{
					rowCurrent.Clear();
					this.Rows[e.RowIndex].Height = rowDefaultHeight;
					this.Rows[e.RowIndex].DividerHeight = rowDefaultDivider;

					this.ClearSelection();
					collapseRow = true;
					this.Rows[e.RowIndex].Selected = true;
					if (EControlType == controlType.middle)
					{
						var oParent = ((MasterControl)this.Parent.Parent);
						oParent.Rows[oParent.ExpandRowIndex].Height = rowDefaultHeight * (this.Rows.Count + 4);
						oParent.Rows[oParent.ExpandRowIndex].DividerHeight = rowDefaultHeight * (this.Rows.Count + 3);
						if (oParent.Rows[oParent.ExpandRowIndex].Height > 500)
						{
							oParent.Rows[oParent.ExpandRowIndex].Height = 500;
							oParent.Rows[oParent.ExpandRowIndex].Height = 480;
						}
					}
				}
				//展开
				else
				{
					if (!(rowCurrent.Count == 0))
					{
						var eRow = rowCurrent[0];
						rowCurrent.Clear();
						this.Rows[eRow].Height = rowDefaultHeight;
						this.Rows[eRow].DividerHeight = rowDefaultDivider;
						this.ClearSelection();
						collapseRow = true;
						this.Rows[eRow].Selected = true;
					}
					rowCurrent.Add(e.RowIndex);
					this.ClearSelection();
					collapseRow = true;
					this.Rows[e.RowIndex].Selected = true;
					this.ExpandRowIndex = e.RowIndex;

					this.Rows[e.RowIndex].Height = 66 + rowDefaultHeight * (((DataView)(childView.childGrid[0].DataSource)).Count + 1);
					this.Rows[e.RowIndex].DividerHeight = 66 + rowDefaultHeight * (((DataView)(childView.childGrid[0].DataSource)).Count);
					//设置一个最大高度
					if (this.Rows[e.RowIndex].Height > 500)
					{
						this.Rows[e.RowIndex].Height = 500;
						this.Rows[e.RowIndex].DividerHeight = 480;
					}
					if (EControlType == controlType.middle)
					{
						if (this.Parent.Parent.GetType() != typeof(MasterControl))
							return;
						var oParent = ((MasterControl)this.Parent.Parent);
						oParent.Rows[oParent.ExpandRowIndex].Height = this.Rows[e.RowIndex].Height + rowDefaultHeight * (this.Rows.Count + 3);
						oParent.Rows[oParent.ExpandRowIndex].DividerHeight = this.Rows[e.RowIndex].DividerHeight + rowDefaultHeight * (this.Rows.Count + 3);
						if (oParent.Rows[oParent.ExpandRowIndex].Height > 500)
						{
							oParent.Rows[oParent.ExpandRowIndex].Height = 500;
							oParent.Rows[oParent.ExpandRowIndex].Height = 480;
						}
					}
					//if (EControlType == controlType.outside)
					//{
					//    //SetControl(this);
					//}
					//this.Rows[e.RowIndex].Height = rowExpandedHeight;
					//this.Rows[e.RowIndex].DividerHeight = rowExpandedDivider;
				}
				//this.ClearSelection();
				//collapseRow = true;
				//this.Rows[e.RowIndex].Selected = true;
			}
			else
			{
				collapseRow = false;
			}
		}
		catch (Exception ex)
		{

		}
	}

	//控件的行重绘事件
	private void MasterControl_RowPostPaint(object obj_sender, DataGridViewRowPostPaintEventArgs e)
	{
		try
		{
			var sender = (DataGridView)obj_sender;
			//set childview control
			var rect = new Rectangle((int)(e.RowBounds.X + ((double)(rowDefaultHeight - 16) / 2)), (int)(e.RowBounds.Y + ((double)(rowDefaultHeight - 16) / 2)), 16, 16);
			if (collapseRow)
			{
				if (this.rowCurrent.Contains(e.RowIndex))
				{
					#region 更改点开后背景色 刘金龙
					var rect1 = new Rectangle(e.RowBounds.X, e.RowBounds.Y + rowDefaultHeight, e.RowBounds.Width, e.RowBounds.Height - rowDefaultHeight);
					using (Brush b = new SolidBrush(Color.FromArgb(164, 169, 143)))
					{
						e.Graphics.FillRectangle(b, rect1);
					}
					#endregion
					sender.Rows[e.RowIndex].DividerHeight = sender.Rows[e.RowIndex].Height - rowDefaultHeight;
					e.Graphics.DrawImage(RowHeaderIconList.Images[(int)rowHeaderIcons.collapse], rect);
					childView.Location = new Point(e.RowBounds.Left + sender.RowHeadersWidth, e.RowBounds.Top + rowDefaultHeight + 5);
					childView.Width = e.RowBounds.Right - sender.RowHeadersWidth;
					childView.Height = System.Convert.ToInt32(sender.Rows[e.RowIndex].DividerHeight - 10);
					childView.Visible = true;
				}
				else
				{
					childView.Visible = false;
					e.Graphics.DrawImage(RowHeaderIconList.Images[(int)rowHeaderIcons.expand], rect);
				}
				collapseRow = false;
			}
			else
			{
				if (this.rowCurrent.Contains(e.RowIndex))
				{
					#region 更改点开后背景色 刘金龙
					var rect1 = new Rectangle(e.RowBounds.X, e.RowBounds.Y + rowDefaultHeight, e.RowBounds.Width, e.RowBounds.Height - rowDefaultHeight);
					using (Brush b = new SolidBrush(Color.FromArgb(164,169,143)))
					{
						e.Graphics.FillRectangle(b, rect1);
					}
					#endregion
					sender.Rows[e.RowIndex].DividerHeight = sender.Rows[e.RowIndex].Height - rowDefaultHeight;
					e.Graphics.DrawImage(RowHeaderIconList.Images[(int)rowHeaderIcons.collapse], rect);
					childView.Location = new Point(e.RowBounds.Left + sender.RowHeadersWidth, e.RowBounds.Top + rowDefaultHeight + 5);
					childView.Width = e.RowBounds.Right - sender.RowHeadersWidth;
					childView.Height = System.Convert.ToInt32(sender.Rows[e.RowIndex].DividerHeight - 10);
					childView.Visible = true;
				}
				else
				{
					childView.Visible = false;
					e.Graphics.DrawImage(RowHeaderIconList.Images[(int)rowHeaderIcons.expand], rect);
				}
			}
			cModule.rowPostPaint_HeaderCount(sender, e);
		}
		catch
		{

		}
	}

	//控件的滚动条滚动事件
	private void MasterControl_Scroll(object sender, ScrollEventArgs e)
	{
		try
		{
			if (!(rowCurrent.Count == 0))
			{
				collapseRow = true;
				this.ClearSelection();
				this.Rows[rowCurrent[0]].Selected = true;
			}
		}
		catch
		{

		}
	}

	//控件的单元格选择事件
	private void MasterControl_SelectionChanged(object sender, EventArgs e)
	{
		try
		{
			if (!(this.RowCount == 0))
			{
				if (rowCurrent.Contains(this.CurrentRow.Index))
				{
					foreach (DataGridView cGrid in childView.childGrid)
					{
						((DataView)cGrid.DataSource).RowFilter = string.Format(_filterFormat, this[_primaryKey, this.CurrentRow.Index].Value);
					}
				}
			}
		}
		catch
		{

		}
	}
	#endregion

	#region Private
	//设置构造函数的参数
	private void SetMasterControl(DataSet cDataset, controlType eControlType)
	{
		//1.控件初始化赋值
		this.Controls.Add(childView);
		InitializeComponent();
		_cDataset = cDataset;
		childView._cDataset = cDataset;
		cModule.applyGridTheme(this);
		Dock = DockStyle.Fill;
		EControlType = eControlType;
		this.AllowUserToAddRows = false;

		//2.通过读取DataSet里面的Relations得到表的关联关系
		if (cDataset.Relations.Count <= 0)
		{
			return;
		}
		DataRelation oRelates;
		if (eControlType == controlType.outside)
		{
			oRelates = cDataset.Relations[1];
			childView.Add(oRelates.ParentTable.TableName, oRelates.ParentColumns[0].ColumnName, oRelates.ChildColumns[0].ColumnName);
		}
		else if (eControlType == controlType.middle)
		{
			oRelates = cDataset.Relations[cDataset.Relations.Count - 1];
			childView.Add2(oRelates.ChildTable.TableName);
		}

		//3.设置主外键对应关系
		oRelates = cDataset.Relations[0];
		//主表里面的值，副表里面的过滤字段
		setParentSource(oRelates.ParentTable.TableName,oRelates.ParentColumns[0].ColumnName, oRelates.ChildColumns[0].ColumnName);
	}

	private void SetControl(MasterControl oGrid)
	{
		oGrid.childView.RemoveControl();
		//oGrid.childView.Controls.RemoveByKey("ChildrenMaster");
		//
		//var oRelates = _cDataset.Relations[1];
		//oGrid.childView.Add(oRelates.ParentTable.TableName, oRelates.ChildColumns[0].ColumnName);


		//foreach (var oGridControl in oGrid.Controls)
		//{
		//    if (oGridControl.GetType() != typeof(detailControl))
		//    {
		//        continue;
		//    }
		//    var DetailControl =(detailControl)oGridControl;
		//    foreach (var odetailControl in DetailControl.Controls)
		//    {
		//        if (odetailControl.GetType() != typeof(MasterControl))
		//        {
		//            continue;
		//        }
		//        var OMasterControl = (MasterControl)odetailControl;
		//        foreach (var oMasterControl in OMasterControl.Controls)
		//        {
		//            if (oMasterControl.GetType() == typeof(detailControl))
		//            {
		//                ((detailControl)oMasterControl).Visible = false;
		//                return;
		//            }
		//        }
		//    }
		//}
	}

	//将List集合转换成DataTable
	private DataTable Fill(object obj)
	{
		if(!(obj is IList))
		{
			return null;
		}
		var objlist = obj as IList;
		if (objlist == null || objlist.Count <= 0)
		{
			return null;
		}
		var tType = objlist[0];
		DataTable dt = new DataTable(tType.GetType().Name);
		DataColumn column;
		DataRow row;
		System.Reflection.PropertyInfo[] myPropertyInfo = tType.GetType().GetProperties(BindingFlags.Public | BindingFlags.Instance);
		foreach (var t in objlist)
		{
			if (t == null)
			{
				continue;
			}
			row = dt.NewRow();
			for (int i = 0, j = myPropertyInfo.Length; i < j; i++)
			{
				System.Reflection.PropertyInfo pi = myPropertyInfo[i];
				string name = pi.Name;
				if (dt.Columns[name] == null)
				{
					column = new DataColumn(name, pi.PropertyType);
					dt.Columns.Add(column);
				}
				row[name] = pi.GetValue(t, null);
			}
			dt.Rows.Add(row);
		}
		return dt;
	}
	#endregion
}
```

2. detailControl 变量作为一个容器用来保存子表格，代码如下：

```csharp
public class detailControl : Ewin.Client.Frame.Controls.EwinPanel
{
	#region 字段
	public List<DataGridView> childGrid = new List<DataGridView>();
	public DataSet _cDataset;
	#endregion

	#region 方法
	public void Add(string tableName, string strPrimaryKey, string strForeignKey)
	{
		//TabPage tPage = new TabPage() { Text = pageCaption };
		//this.Controls.Add(tPage);
		var newGrid = new MasterControl(_cDataset, controlType.middle) { Dock = DockStyle.Fill, DataSource = new DataView(_cDataset.Tables[tableName]) };
		newGrid.setParentSource(tableName, strPrimaryKey, strForeignKey);//设置主外键
		//newGrid.Name = "ChildrenMaster";
		//tPage.Controls.Add(newGrid);
		this.Controls.Add(newGrid);
		//this.BorderStyle = BorderStyle.FixedSingle;
		cModule.applyGridTheme(newGrid);
		cModule.setGridRowHeader(newGrid);
		newGrid.RowPostPaint += cModule.rowPostPaint_HeaderCount;
		childGrid.Add(newGrid);
	}

	public void Add2(string tableName)
	{
		//TabPage tPage = new TabPage() { Text = pageCaption };
		//this.Controls.Add(tPage);
		DataGridView newGrid = new Ewin.Client.Frame.Controls.EwinGrid() { Dock = DockStyle.Fill, DataSource = new DataView(_cDataset.Tables[tableName]) };
		newGrid.AllowUserToAddRows = false;
		//tPage.Controls.Add(newGrid);
		this.Controls.Add(newGrid);
		cModule.applyGridTheme(newGrid);
		cModule.setGridRowHeader(newGrid);
		newGrid.RowPostPaint += cModule.rowPostPaint_HeaderCount;
		childGrid.Add(newGrid);
	}

	public void RemoveControl()
	{
		this.Controls.Remove(childGrid[0]);
		childGrid.Clear();
	}
	#endregion

}
```

3. cModule.cs 用来设置样式

```csharp
namespace Ewin.Client.Frame.UcGrid
{
    /// <summary>
    /// 折叠控件样式以及行数操作类
    /// </summary>
    sealed class cModule
    {
        #region CustomGrid
        static System.Windows.Forms.DataGridViewCellStyle dateCellStyle = new System.Windows.Forms.DataGridViewCellStyle { Alignment = DataGridViewContentAlignment.MiddleRight };
        static System.Windows.Forms.DataGridViewCellStyle amountCellStyle = new System.Windows.Forms.DataGridViewCellStyle { Alignment = DataGridViewContentAlignment.MiddleRight, Format = "N2" };
        static System.Windows.Forms.DataGridViewCellStyle gridCellStyle = new System.Windows.Forms.DataGridViewCellStyle
        {
            Alignment = System.Windows.Forms.DataGridViewContentAlignment.MiddleLeft,
            BackColor = System.Drawing.Color.FromArgb(System.Convert.ToInt32(System.Convert.ToByte(79)), System.Convert.ToInt32(System.Convert.ToByte(129)), System.Convert.ToInt32(System.Convert.ToByte(189))),
            Font = new System.Drawing.Font("Segoe UI", (float)(10.0F), System.Drawing.FontStyle.Regular, System.Drawing.GraphicsUnit.Point, System.Convert.ToByte(0)),
            ForeColor = System.Drawing.SystemColors.ControlLightLight,
            SelectionBackColor = System.Drawing.SystemColors.Highlight,
            SelectionForeColor = System.Drawing.SystemColors.HighlightText,
            WrapMode = System.Windows.Forms.DataGridViewTriState.True
        };
        static System.Windows.Forms.DataGridViewCellStyle gridCellStyle2 = new System.Windows.Forms.DataGridViewCellStyle
        {
            Alignment = System.Windows.Forms.DataGridViewContentAlignment.MiddleLeft,
            BackColor = System.Drawing.SystemColors.ControlLightLight,
            Font = new System.Drawing.Font("Segoe UI", (float)(10.0F), System.Drawing.FontStyle.Regular, System.Drawing.GraphicsUnit.Point, System.Convert.ToByte(0)),
            ForeColor = System.Drawing.SystemColors.ControlText,
            SelectionBackColor = System.Drawing.Color.FromArgb(System.Convert.ToInt32(System.Convert.ToByte(155)), System.Convert.ToInt32(System.Convert.ToByte(187)), System.Convert.ToInt32(System.Convert.ToByte(89))),
            SelectionForeColor = System.Drawing.SystemColors.HighlightText,
            WrapMode = System.Windows.Forms.DataGridViewTriState.False
        };
        static System.Windows.Forms.DataGridViewCellStyle gridCellStyle3 = new System.Windows.Forms.DataGridViewCellStyle
        {
            Alignment = System.Windows.Forms.DataGridViewContentAlignment.MiddleLeft,
            BackColor = System.Drawing.Color.WhiteSmoke,
            Font = new System.Drawing.Font("Segoe UI", (float)(10.0F), System.Drawing.FontStyle.Regular, System.Drawing.GraphicsUnit.Point, System.Convert.ToByte(0)),
            ForeColor = System.Drawing.SystemColors.WindowText,
            SelectionBackColor = System.Drawing.Color.FromArgb(System.Convert.ToInt32(System.Convert.ToByte(155)), System.Convert.ToInt32(System.Convert.ToByte(187)), System.Convert.ToInt32(System.Convert.ToByte(89))),
            SelectionForeColor = System.Drawing.SystemColors.HighlightText,
            WrapMode = System.Windows.Forms.DataGridViewTriState.True
        };

        //设置表格的主题样式
        static public void applyGridTheme(DataGridView grid)
        {
            grid.AllowUserToAddRows = false;
            grid.AllowUserToDeleteRows = false;
            grid.BackgroundColor = System.Drawing.SystemColors.Window;
            grid.BorderStyle = System.Windows.Forms.BorderStyle.None;
            grid.ColumnHeadersBorderStyle = System.Windows.Forms.DataGridViewHeaderBorderStyle.Single;
            grid.ColumnHeadersDefaultCellStyle = gridCellStyle;
            grid.ColumnHeadersHeight = 32;
            grid.ColumnHeadersHeightSizeMode = System.Windows.Forms.DataGridViewColumnHeadersHeightSizeMode.DisableResizing;
            grid.DefaultCellStyle = gridCellStyle2;
            grid.EnableHeadersVisualStyles = false;
            grid.GridColor = System.Drawing.SystemColors.GradientInactiveCaption;
            //grid.ReadOnly = true;
            grid.RowHeadersVisible = true;
            grid.RowHeadersBorderStyle = System.Windows.Forms.DataGridViewHeaderBorderStyle.Single;
            grid.RowHeadersDefaultCellStyle = gridCellStyle3;
            grid.Font = gridCellStyle.Font;
        }

        //设置表格单元格样式
        static public void setGridRowHeader(DataGridView dgv, bool hSize = false)
        {
            dgv.TopLeftHeaderCell.Value = "NO ";
            dgv.TopLeftHeaderCell.Style.Alignment = DataGridViewContentAlignment.MiddleCenter;
            dgv.AutoResizeRowHeadersWidth(DataGridViewRowHeadersWidthSizeMode.AutoSizeToDisplayedHeaders);
            foreach (DataGridViewColumn cCol in dgv.Columns)
            {
                if (cCol.ValueType.ToString() == typeof(DateTime).ToString())
                {
                    cCol.DefaultCellStyle = dateCellStyle;
                }
                else if (cCol.ValueType.ToString() == typeof(decimal).ToString() || cCol.ValueType.ToString() == typeof(double).ToString())
                {
                    cCol.DefaultCellStyle = amountCellStyle;
                }
            }
            if (hSize)
            {
                dgv.RowHeadersWidth = dgv.RowHeadersWidth + 16;
            }
            dgv.AutoResizeColumns();
        }

        //设置表格的行号
        static public void rowPostPaint_HeaderCount(object obj_sender, DataGridViewRowPostPaintEventArgs e)
        {
            try
            {
                var sender = (DataGridView)obj_sender;
                //set rowheader count
                DataGridView grid = (DataGridView)sender;
                string rowIdx = System.Convert.ToString((e.RowIndex + 1).ToString());
                var centerFormat = new StringFormat();
                centerFormat.Alignment = StringAlignment.Center;
                centerFormat.LineAlignment = StringAlignment.Center;
                Rectangle headerBounds = new Rectangle(e.RowBounds.Left, e.RowBounds.Top,
                    grid.RowHeadersWidth, e.RowBounds.Height - sender.Rows[e.RowIndex].DividerHeight);
                e.Graphics.DrawString(rowIdx, grid.Font, SystemBrushes.ControlText,
                    headerBounds, centerFormat);
            }
            catch (Exception ex)
            {

            }
        }
        #endregion
    }

    /// <summary>
    /// 控件类型，是最外层的表格还是中间层的表格
    /// </summary>
    public enum controlType
    {
        outside = 0,
        middle = 1
    }

    /// <summary>
    /// 展开图标
    /// </summary>
    public enum rowHeaderIcons
    {
        expand = 0,
        collapse = 1
    }
}
```

4. From 页面调用

```text
#region 使用方法一
//var oDataSet = GetDataSet();
//
//masterDetail = new MasterControl(oDataSet, controlType.outside);
#endregion

#region 使用方法二
var dicRelateData1 = new Dictionary<string, string>();
var dicRelateData2 = new Dictionary<string, string>();
dicRelateData1.Add("Menu_ID","Menu_ID");//表格一和表格二之间的主外键关系
dicRelateData2.Add("Menu_Name2","Menu_Name2");//表格二和表格三之间的主外键关系
masterDetail = new MasterControl(GetDataSource(), GetDataSource2(), GetDataSource3(), dicRelateData1, dicRelateData2, controlType.outside);
#endregion

panelView.Controls.Add(masterDetail);
```

 昨天应领导要求，折叠控件增加了折叠线的效果，看起来有没有更加像模像样了。~~~

其实就在行重绘事件 `private void MasterControl_RowPostPaint(object obj_sender, DataGridViewRowPostPaintEventArgs e)` 里面增加了如下代码：

```csharp
using (Pen p = new Pen(Color.GhostWhite))
{
	var iHalfWidth = (e.RowBounds.Left + sender.RowHeadersWidth) / 2;
	var oPointHLineStart = new Point(rect1.X + iHalfWidth, rect1.Y);
	var oPointHLineEnd = new Point(rect1.X + iHalfWidth, rect1.Y + rect1.Height / 2);
	e.Graphics.DrawLine(p, oPointHLineStart, oPointHLineEnd);
	//折叠线
	e.Graphics.DrawLine(p, oPointHLineEnd, new Point(oPointHLineEnd.X + iHalfWidth, oPointHLineEnd.Y));
}
```

效果如下：
![[好看的DataGridView折叠控件-5.png]]

2015-07-01

PS：原以为 CS 的控件大家不会太感兴趣，这两天很多园友找我要源码，其实并非舍不得将源码共享，只是很多东西融入到项目中了很难分离出来，需要时间整理，望理解。知道这么多园友对 CS 的控件也感兴趣，我就抽时间整理了下折叠控件的 Demo。本着大家共同进步的原则将源码共享出来。好的东西就要共享，咱.Net 也要慢慢走共享开源路线哈

[源码下载](https://files.cnblogs.com/files/landeanfen/DataGridview%E6%8A%98%E5%8F%A0%E6%8E%A7%E4%BB%B6.rar)
