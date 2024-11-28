
# 基于NotePad应用实现功能扩展

## 1.基本功能

-   NoteList界面中笔记条目增加时间戳显示

1.  **修改布局文件**：在 `noteslist_item.xml` 中添加一个新的 `TextView` 用于显示时间戳，可以将其放置在原有 `TextView` 的下方。
	
	```
	<!--添加一个垂直的显示布局-->  
	<LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"  
	    android:id="@+id/layout"  
	    android:layout_width="match_parent"  
	    android:layout_height="match_parent"  
	    android:orientation="vertical">  

	    <TextView xmlns:android="http://schemas.android.com/apk/res/android"  
	        android:id="@android:id/text1"  
	        android:layout_width="match_parent"  
	        android:layout_height="?android:attr/listPreferredItemHeight"  
	        android:textAppearance="?android:attr/textAppearanceLarge"  
	        android:gravity="center_vertical"  
	        android:paddingLeft="5dip"  
	        android:singleLine="true" />  

	    <!--添加 显示时间 的TextView-->  
	    <TextView  
	        android:id="@+id/timestamp"  
	        android:layout_width="match_parent"  
	        android:layout_height="wrap_content"  
	        android:textAppearance="?android:attr/textAppearanceSmall"  
	        android:paddingLeft="5dip" />  

	</LinearLayout>
	```

2.  **修改`NotesList.java`中**PROJECTION**的内容**：添加**modif**字段，使其在后面的搜索中才能从**SQLite**中读取修改时间的字段。
    
    ```
    private static final String[] PROJECTION = new String[] {  
        NotePad.Notes._ID, // 0  
        NotePad.Notes.COLUMN_NAME_TITLE, // 1  
        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE  
    };
    ```
    
3.  **在dataColumns，viewIDs中补充时间部分**：修改适配器内容，增加**dataColumns**中装配到**ListView**的内容。
    
    ```
    String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE , NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;  
    int[] viewIDs = { android.R.id.text1, R.id.timestamp };
    ```
    
4.  **更新 `SimpleCursorAdapter`**：通过自定义`SimpleCursorAdapter.ViewBinder`更改数据显示格式，将时间戳格式化。
    
    ```
    // Creates the backing adapter for the ListView.  
    SimpleCursorAdapter adapter = new SimpleCursorAdapter(  
        this,                             // The Context for the ListView  
        R.layout.noteslist_item,          // Points to the XML for a list item  
        cursor,                           // The cursor to get items from  
        dataColumns,                      // 这里需要包含时间戳的列名  
        viewIDs  
    );  
    // 如果需要格式化时间戳，可以在适配器中重写 bindView 方法  
    adapter.setViewBinder(new SimpleCursorAdapter.ViewBinder() {  
    @Override  
    public boolean setViewValue(View view, Cursor cursor, int columnIndex) {  
        if (view.getId() == R.id.timestamp) {  
            long timestamp = cursor.getLong(columnIndex);  
            // 格式化时间戳为可读格式  
            String formattedDate = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault()).format(new Date(timestamp));  
            ((TextView) view).setText(formattedDate);  
            return true; // 表示我们处理了这个视图  
        }  
        return false; // 让适配器处理其他视图  
    }  
    });
    ```
    
5.  **运行效果**
	在标题列表下方显示编辑的时间：
	
    ![image](https://github.com/user-attachments/assets/cf99097a-ba00-4a7a-871c-7698dd295352)



-   添加笔记查询功能（根据标题或内容查询）

1.  **更新菜单资源文件**：在 `list_options_menu.xml` 中添加一个搜索的item，搜索图标用安卓自带的图标，设为总是显示。

	```
	<item  
	    android:id="@+id/menu_search"  
	    android:icon="@android:drawable/ic_menu_search"  
	    android:title="@string/menu_search"  
	    android:showAsAction="always" >  
	</item>
	```
	
2.  **创建搜索界面**：在 `res/layout` 目录下创建 `note_search.xml` 文件，内容如下：
	```
	<?xml version="1.0" encoding="utf-8"?>  
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
	    android:orientation="vertical" android:layout_width="match_parent"  
	    android:layout_height="match_parent">  
	  
	    <SearchView        
		    android:id="@+id/search_view"  
	        android:layout_width="match_parent"  
	        android:layout_height="wrap_content"  
	        android:iconifiedByDefault="false"  
	        android:queryHint="输入标题或内容进行搜索"  
	        android:layout_alignParentTop="true">  
	    </SearchView>  
	    <ListView        
		    android:id="@android:id/list"  
	        android:layout_width="match_parent"  
	        android:layout_height="wrap_content">  
	    </ListView>  
	</LinearLayout>
	```
	
3.  **创建MyCursorAdapter**：确保有一个自定义的 `MyCursorAdapter` 类来处理 `Cursor` 数据并将其绑定到 `ListView`。在这个适配器中，将格式化时间戳并将其显示在相应的 `TextView` 中：

	```
	package com.example.android.notepad;  
	  
	import android.annotation.SuppressLint;  
	import android.content.Context;  
	import android.database.Cursor;  
	import android.view.LayoutInflater;  
	import android.view.View;  
	import android.view.ViewGroup;  
	import android.widget.SimpleCursorAdapter;  
	import android.widget.TextView;  
	  
	import java.text.SimpleDateFormat;  
	import java.util.Date;  
	import java.util.Locale;  
	  
	public class MyCursorAdapter extends SimpleCursorAdapter {  
	    private LayoutInflater inflater;  
	  
	    public MyCursorAdapter(Context context, int layout, Cursor c, String[] from, int[] to) {  
	        super(context, layout, c, from, to, 0);  
	        inflater = LayoutInflater.from(context);  
	    }  
	  
	    @Override  
	    public View getView(int position, View convertView, ViewGroup parent) {  
	        View view = super.getView(position, convertView, parent);  
	  
	        // 获取 Cursor        
	        Cursor cursor = getCursor();  
	        if (cursor != null && cursor.moveToPosition(position)) {  
	            // 获取时间戳  
	            @SuppressLint("Range") long timestamp = cursor.getLong(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE));  
	            // 格式化时间戳  
	            String formattedDate = formatDate(timestamp);  
	            // 设置格式化后的日期到 TextView            TextView timestampView = view.findViewById(R.id.timestamp);  
	            timestampView.setText(formattedDate);  
	        }  
	  
	        return view;  
	    }  
	  
	    private String formatDate(long timestamp) {  
	        // 创建 SimpleDateFormat 实例  
	        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault());  
	        return sdf.format(new Date(timestamp));  
	    }  
	}
	```
	
4.  **创建 `NoteSearch.java`**：在 `com.example.android.notepad` 包中创建 `NoteSearch.java` 文件，内容如下：

	```
	package com.example.android.notepad;  
	  
	import android.app.ListActivity;  
	import android.content.ContentUris;  
	import android.content.Intent;  
	import android.database.Cursor;  
	import android.net.Uri;  
	import android.os.Bundle;  
	import android.view.View;  
	import android.widget.ListView;  
	import android.widget.SearchView;  
	  
	public class NoteSearch extends ListActivity implements SearchView.OnQueryTextListener {  
	    private static final String[] PROJECTION = new String[] {  
	            NotePad.Notes._ID, // 0  
	            NotePad.Notes.COLUMN_NAME_TITLE, // 1  
	            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE // 2  
	    };  
	  
	    @Override  
	    protected void onCreate(Bundle savedInstanceState) {  
	        super.onCreate(savedInstanceState);  
	        setContentView(R.layout.note_search);  
	  
	        // 获取 Intent 数据  
	        Intent intent = getIntent();  
	        if (intent.getData() == null) {  
	            intent.setData(NotePad.Notes.CONTENT_URI);  
	        }  
	  
	        // 初始化 SearchView        
	        SearchView searchView = findViewById(R.id.search_view);  
	        searchView.setOnQueryTextListener(this);  
	    }  
	  
	    @Override  
	    public boolean onQueryTextSubmit(String query) {  
	        return false; // 不处理提交事件  
	    }  
	  
	    @Override  
	    public boolean onQueryTextChange(String newText) {  
	        // 构建查询条件  
	        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " LIKE ? OR " +  
	                NotePad.Notes.COLUMN_NAME_NOTE + " LIKE ?";  
	        String[] selectionArgs = new String[]{"%" + newText + "%", "%" + newText + "%"};  
	  
	        // 查询数据库  
	        Cursor cursor = getContentResolver().query(  
	                getIntent().getData(), // 使用默认内容 URI  
	                PROJECTION, // 返回的列  
	                selection, // 查询条件  
	                selectionArgs, // 查询参数  
	                NotePad.Notes.DEFAULT_SORT_ORDER // 默认排序  
	        );  
	  
	        // 创建适配器并设置给 ListView        
	        String[] dataColumns = {NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE};  
	        int[] viewIDs = {android.R.id.text1, R.id.timestamp};  
	        MyCursorAdapter adapter = new MyCursorAdapter(  
	                this,  
	                R.layout.noteslist_item,  
	                cursor,  
	                dataColumns,  
	                viewIDs  
	        );  
	        setListAdapter(adapter);  
	        return true;  
	    }  
	  
	    @Override  
	    protected void onListItemClick(ListView l, View v, int position, long id) {  
	        // 构建 URI        
	        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);  
	        String action = getIntent().getAction();  
	  
	        // 处理点击事件  
	        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {  
	            setResult(RESULT_OK, new Intent().setData(uri));  
	        } else {  
	            // 启动编辑 Activity            
	            startActivity(new Intent(Intent.ACTION_EDIT, uri));  
	        }  
	    }  
	}
	```
	
5. **处理搜索菜单项的选择**：在 `NotesList.java` 中的`onOptionsItemSelected` 方法中处理搜索菜单项的点击事件：

	```
	case R.id.menu_search:  
	    // 启动搜索 Activity    
	    startActivity(new Intent(this, NoteSearch.class));  
	    return true;
	```
	
6. **更新 AndroidManifest.xml**：在 `AndroidManifest.xml` 中注册新的 `NoteSearch` Activity：

	```
	<!--添加搜索activity-->  
	<activity  
	    android:name="NoteSearch"  
	    android:label="@string/title_notes_search">  
	</activity>
	```
	
7. **运行效果**
	输入内容进行查询，会动态显示包含该内容的笔记列表：

  ![image](https://github.com/user-attachments/assets/35602936-2a11-4faf-b3fd-ece13e62e1ef)

	
	
## 2.附加功能

 -  UI美化
 
 1. **更换主题为白色**：
	 在`AndroidManifest.xml`中`NotesList`的Activity中添加：
	```
	android:theme="@android:style/Theme.Holo.Light"
	```
	更换后效果图：
	
  ![image](https://github.com/user-attachments/assets/773038db-545d-48b2-88f2-4d8bd068c9fb)


2. **修改`NotePad.java`**：

	在 `NotePad` 契约类中添加背景颜色的列名：
	```
	public static final String COLUMN_NAME_BACK_COLOR = "color";
	```
	在 `NotePad` 契约类中定义颜色常量：
	```
	public static final int DEFAULT_COLOR = 0; //白
	public static final int YELLOW_COLOR = 1; //黄
	public static final int BLUE_COLOR = 2; //蓝
	public static final int GREEN_COLOR = 3; //绿
	public static final int RED_COLOR = 4; //红
	```

3.  **修改`NotePadProvider`**：

	在 `NotePadProvider` 中创建数据库表的地方添加颜色字段：
	```
	 @Override
	    public void onCreate(SQLiteDatabase db) {
	        db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + "   ("
	        + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
	        + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
	        + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
	        + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
	        + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
	        + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER" //颜色
	        + ");");
	       }
	```
	在 `NotePadProvider` 的 `static {}` 块中添加颜色字段的映射:
	```
	sNotesProjectionMap.put(
	        NotePad.Notes.COLUMN_NAME_BACK_COLOR,
	        NotePad.Notes.COLUMN_NAME_BACK_COLOR);
	```
	在 `NotePadProvider` 的 `insert` 方法中添加默认颜色设置：
	```
	 // 新建笔记，背景默认为白色
	if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
	        values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
	        }
	```
4. **修改`MyCursorAdapter`**：
	自定义一个 `CursorAdapter` 继承自 `SimpleCursorAdapter`，并重写 `bindView` 方法：
	```
	@Override  
	public void bindView(View view, Context context, Cursor cursor){  
	    super.bindView(view, context, cursor);  
	    //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色  
	    @SuppressLint("Range") int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));  
	    /**  
	     * 白 255 255 255  
	     * 黄 247 216 133  
	     * 蓝 165 202 237  
	     * 绿 161 214 174  
	     * 红 244 149 133  
	     */    switch (x){  
	        case NotePad.Notes.DEFAULT_COLOR:  
	            view.setBackgroundColor(Color.rgb(255, 255, 255));  
	            break;  
	        case NotePad.Notes.YELLOW_COLOR:  
	            view.setBackgroundColor(Color.rgb(247, 216, 133));  
	            break;  
	        case NotePad.Notes.BLUE_COLOR:  
	            view.setBackgroundColor(Color.rgb(165, 202, 237));  
	            break;  
	        case NotePad.Notes.GREEN_COLOR:  
	            view.setBackgroundColor(Color.rgb(161, 214, 174));  
	            break;  
	        case NotePad.Notes.RED_COLOR:  
	            view.setBackgroundColor(Color.rgb(244, 149, 133));  
	            break;  
	        default:  
	            view.setBackgroundColor(Color.rgb(255, 255, 255));  
	            break;  
	    }  
	}
	```
	 在 `bindView` 方法中，根据从数据库中获取的颜色值设置每个列表项的背景颜色。这样，列表中的每个笔记项都能显示其对应的背景颜色。
	
5. **修改`NoteList`**：
	在 `NoteList` 中的 `PROJECTION` 数组中添加颜色项：
	```
	private static final String[] PROJECTION = new String[] {  
	        NotePad.Notes._ID, // 0  
	        NotePad.Notes.COLUMN_NAME_TITLE, // 1  
	        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//2  
	        NotePad.Notes.COLUMN_NAME_BACK_COLOR,//3  
	};
	```
	
	将 `adapter`、`cursor`、`dataColumns` 和 `viewIDs` 定义在类内：
		
	```
	private MyCursorAdapter adapter;
	private Cursor cursor;
	private String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
	private int[] viewIDs = { android.R.id.text1 , R.id.text1_time };
	```
	将适配器和数据列的定义移到类的成员变量中，以便在多个方法中使用。
	
	将 `NoteList` 中的 `SimpleCursorAdapter` 替换为 `MyCursorAdapter`：
	```
	 //修改为可以填充颜色的自定义的adapter
	adapter = new MyCursorAdapter(
	        this,
	        R.layout.noteslist_item,
	        cursor,
	        dataColumns,
	        viewIDs
	    );
	```
6. **运行效果**：
	通过以上修改，用户在笔记列表中可以看到每个笔记项的背景颜色与其在编辑器中设置的颜色一致。
	
  ![image](https://github.com/user-attachments/assets/7ea2d220-d584-4b94-91ed-020d081903df)



- 背景更换

1.	**编辑`NoteEditor`**：
	在 `NoteEditor` 类中定义一个 `PROJECTION` 数组，以便在查询笔记时包含背景颜色的信息：
	
	```
	  private static final String[] PROJECTION =
	        new String[] {
	            NotePad.Notes._ID,
	            NotePad.Notes.COLUMN_NAME_TITLE,
	            NotePad.Notes.COLUMN_NAME_NOTE,
	            NotePad.Notes.COLUMN_NAME_BACK_COLOR
	    };
	```
	
	在 `NoteEditor` 的 `onResume()` 方法中读取颜色数据并设置背景色：
	
	```
	//读取颜色数据
	int x = mCursor.getInt(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
	    /**
	    * 白 255 255 255
	    * 黄 247 216 133
	    * 蓝 165 202 237
	    * 绿 161 214 174
	    * 红 244 149 133
	    */
	    switch (x){
	        case NotePad.Notes.DEFAULT_COLOR:
	            mText.setBackgroundColor(Color.rgb(255, 255, 255));
	            break;
	        case NotePad.Notes.YELLOW_COLOR:
	            mText.setBackgroundColor(Color.rgb(247, 216, 133));
	            break;
	        case NotePad.Notes.BLUE_COLOR:`这里输入代码`
	            mText.setBackgroundColor(Color.rgb(165, 202, 237));
	            break;
	        case NotePad.Notes.GREEN_COLOR:
	            mText.setBackgroundColor(Color.rgb(161, 214, 174));
	            break;
	        case NotePad.Notes.RED_COLOR:
	            mText.setBackgroundColor(Color.rgb(244, 149, 133));
	            break;
	        default:
	            mText.setBackgroundColor(Color.rgb(255, 255, 255));
	            break;
	    }
	```
  	-   首先，通过 `mCursor.getInt(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR))` 获取当前笔记的背景颜色值。
	-   然后，使用 `switch` 语句根据颜色值设置 `mText` 的背景颜色。每种颜色对应一个 RGB 值，使用 `Color.rgb()` 方法将其转换为颜色对象。
	-   如果颜色值不匹配任何已定义的颜色，则默认设置为白色背景。

2. **更改editor_options_menu.xml**：
	 在菜单文件中添加一个更改背景的选项：
	```
	<item android:id="@+id/menu_color"
	        android:title="@string/menu_color"
	        android:icon="@drawable/ic_menu_color"
	        android:showAsAction="always"/>
	```
	 这段代码在在编辑器的选项菜单中添加了一个新的菜单项，用于更改背景颜色。`android:showAsAction="always"` 表示该菜单项始终显示在操作栏上。
	  
	 在 `NoteEditor` 中找到 `onOptionsItemSelected()` 方法，并添加以下代码：
	```
	//换背景颜色选项
	    case R.id.menu_color:
	        changeColor();
	        break;
	```
    当用户选择更改颜色的菜单项时，调用 `changeColor()` 方法。
   
	在 `NoteEditor` 中添加 `changeColor()` 方法：
	```
	//跳转改变颜色的activity，将uri信息传到新的activity
	    private final void changeColor() {
	        Intent intent = new Intent(null,mUri);
	        intent.setClass(NoteEditor.this,NoteColor.class);
	        NoteEditor.this.startActivity(intent);
	    }
	```
	 该方法创建一个新的 `Intent`，用于启动 `NoteColor` 活动，并将当前笔记的 URI 传递给它。
	
3. **新建布局colors.xml**：
	定义了一组颜色资源，供应用程序使用。这些颜色将用于更改笔记的背景色。
	```
	<?xml version="1.0" encoding="utf-8"?>  
	<resources>  
	    <color name="colorBlack">#000000</color>  
	    <color name="colorYellow">#f7d885</color>  
	    <color name="colorBlue">#a5caed</color>  
	    <color name="colorGreen">#a1d6ae</color>  
	    <color name="colorRed">#f49585</color>  
	    <color name="colorWhite">#FFFFFF</color>  
	</resources>
	```
5. **新建布局note_color.xml**：
	创建了一个水平排列的 `LinearLayout`，其中包含多个 `ImageButton`，每个按钮代表一种颜色。点击按钮时会调用相应的颜色选择方法。
	```
	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:orientation="horizontal" android:layout_width="match_parent"
	    android:layout_height="match_parent">
	    <ImageButton
	        android:id="@+id/color_white"
	        android:layout_width="0dp"
	        android:layout_height="50dp"
	        android:layout_weight="1"
	        android:background="@color/colorWhite"
	        android:onClick="white"/>
	    <ImageButton
	        android:id="@+id/color_yellow"
	        android:layout_width="0dp"
	        android:layout_height="50dp"
	        android:layout_weight="1"
	        android:background="@color/colorYellow"
	        android:onClick="yellow"/>
	    <ImageButton
	        android:id="@+id/color_blue"
	        android:layout_width="0dp"
	        android:layout_height="50dp"
	        android:layout_weight="1"
	        android:background="@color/colorBlue"
	        android:onClick="blue"/>
	    <ImageButton
	        android:id="@+id/color_green"
	        android:layout_width="0dp"
	        android:layout_height="50dp"
	        android:layout_weight="1"
	        android:background="@color/colorGreen"
	        android:onClick="green"/>
	    <ImageButton
	        android:id="@+id/color_red"
	        android:layout_width="0dp"
	        android:layout_height="50dp"
	        android:layout_weight="1"
	        android:background="@color/colorRed"
	        android:onClick="red"/>
	</LinearLayout>
	```
6. **创建NoteColor的Acitvity**：
	`NoteColor` 类用于处理颜色选择。它从 `NoteEditor` 接收 URI，并查询当前笔记的背景颜色。在 `onPause()` 方法中，将选择的颜色更新到数据库中。
	```
	public class NoteColor extends Activity {
	    private Cursor mCursor;
	    private Uri mUri;
	    private int color;
	    private static final int COLUMN_INDEX_TITLE = 1;
	    private static final String[] PROJECTION = new String[] {
	            NotePad.Notes._ID, // 0
	            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
	    };
	    public void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.note_color);
	        //从NoteEditor传入的uri
	        mUri = getIntent().getData();
	        mCursor = managedQuery(
	                mUri,        // The URI for the note that is to be retrieved.
	                PROJECTION,  // The columns to retrieve
	                null,        // No selection criteria are used, so no where columns are needed.
	                null,        // No where columns are used, so no where values are needed.
	                null         // No sort order is needed.
	        );
	    }
	    @Override
	    protected void onResume(){
	    //执行顺序在onCreate之后
	        if (mCursor != null) {
	            mCursor.moveToFirst();
	            color = mCursor.getInt(COLUMN_INDEX_TITLE);
	        }
	        super.onResume();
	    }
	    @Override
	    protected void onPause() {
	    //执行顺序在finish()之后，将选择的颜色存入数据库
	        super.onPause();
	        ContentValues values = new ContentValues();
	        values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, color);
	        getContentResolver().update(mUri, values, null, null);
	    }
	    public void white(View view){
	        color = NotePad.Notes.DEFAULT_COLOR;
	        finish();
	    }
	    public void yellow(View view){
	        color = NotePad.Notes.YELLOW_COLOR;
	        finish();
	    }
	    public void blue(View view){
	        color = NotePad.Notes.BLUE_COLOR;
	        finish();
	    }
	    public void green(View view){
	        color = NotePad.Notes.GREEN_COLOR;
	        finish();
	    }
	    public void red(View view){
	        color = NotePad.Notes.RED_COLOR;
	        finish();
	    }

	}
	``` 
7. **修改AndroidManifest.xml**：
	将 `NoteColor` 活动的主题设置为对话框样式，使其以对话框的形式显示，便于用户选择颜色
	```
	<!--换背景色-->
	<activity android:name="NoteColor"
	    android:theme="@android:style/Theme.Holo.Light.Dialog"
	    android:label="ChangeColor"
	    android:windowSoftInputMode="stateVisible"/>
	```
8. **运行效果**
	 进入编辑页面，点击右上角“皮肤”按钮可更换背景颜色，让编辑笔记时的背景色跟笔记列表的该笔记背景色同为一种颜色。
	如图所示：
 
  ![image](https://github.com/user-attachments/assets/d825693f-7718-43a2-834c-1c1ee0168e04)



-   笔记排序
 
1. **修改文件list_options_menu.xml**：
	在菜单文件list_options_menu.xml中添加：
	在笔记列表的菜单中添加了一个排序选项，包含三个子菜单项，分别用于不同的排序方式。
	```
	<item
	    android:id="@+id/menu_sort"
	    android:title="@string/menu_sort"
	    android:icon="@android:drawable/ic_menu_sort_by_size"
	    android:showAsAction="always" >
	    <menu>
	        <item
	            android:id="@+id/menu_sort1"
	            android:title="@string/menu_sort1"/>
	        <item
	            android:id="@+id/menu_sort2"
	            android:title="@string/menu_sort2"/>
	        <item
	            android:id="@+id/menu_sort3"
	            android:title="@string/menu_sort3"/>
	        </menu>
	    </item>
	```
	
2. **修改NoteList.java**：

	在NoteList菜单switch下添加case：
	根据用户选择的排序方式，查询笔记数据并更新适配器，以显示排序后的笔记列表。
	```
		//创建时间排序
	    case R.id.menu_sort1:
	        cursor = managedQuery(
	                getIntent().getData(),            
	                PROJECTION,                      
	                null,                          
	                null,                          
	                NotePad.Notes._ID 
	                );
	        adapter = new MyCursorAdapter(
	                this,
	                R.layout.noteslist_item,
	                cursor,
	                dataColumns,
	                viewIDs
	        );
	        setListAdapter(adapter);
	        return true;
		//修改时间排序
	    case R.id.menu_sort2:
	        cursor = managedQuery(
	                getIntent().getData(),          
	                PROJECTION,                      
	                null,                            
	                null,                       
	                NotePad.Notes.DEFAULT_SORT_ORDER 
	        );
	        adapter = new MyCursorAdapter(
	                this,
	                R.layout.noteslist_item,
	                cursor,
	                dataColumns,
	                viewIDs
	        );
	        setListAdapter(adapter);
	        return true;
	    //颜色排序
	    case R.id.menu_sort3:
	        cursor = managedQuery(
	                getIntent().getData(),
	                PROJECTION,      
	                null,       
	                null,       
	                NotePad.Notes.COLUMN_NAME_BACK_COLOR
	                );
	        adapter = new MyCursorAdapter(
	                this,
	                R.layout.noteslist_item,
	                cursor,
	                dataColumns,
	                viewIDs
	                );
	        setListAdapter(adapter);
	        return true;
	```

3. **运行效果**
	点击排序图标，可根据创建时间、修改时间、颜色进行笔记列表的排序，如图所示：

  ![image](https://github.com/user-attachments/assets/1e5b9b10-7179-4142-b687-6a5122a6490a)



