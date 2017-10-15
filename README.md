# 3fieldscode

package com.example.manasi.insertintodb3fields;

import android.database.Cursor;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    DB_Controller db;
    private EditText e1,e2,e3;
    private TextView t1;
    String item;
    String cost,qty;
    int sum = 0;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        db=new DB_Controller(this);
        e1 = (EditText)findViewById(R.id.item);
        e2 = (EditText)findViewById(R.id.cost);
        e3 = (EditText)findViewById(R.id.qty);
        t1 = (TextView)findViewById(R.id.result);
    }

    public void insert(View v) {
        item = e1.getText().toString();
        cost = e2.getText().toString();
        qty = e3.getText().toString();

        db.insertitem(item,cost,qty);

        e1.setText("");
        e2.setText("");
        e3.setText("");
        Toast.makeText(getApplicationContext(),"Here",Toast.LENGTH_SHORT).show();
    }

    public void list(View v) {
        t1.setText("");
        Cursor c = db.listitem();
        while(c.moveToNext()) {
            String s= c.getString(0)+ "    " + c.getString(1) + "      " + c.getString(2) + "\n";
            t1.append(s);
        }
    }

    public void getCost(View v) {
        t1.setText("");
        item = e1.getText().toString();
        String s = db.get_cost(item);
        int c = Integer.valueOf(s).intValue();
        sum+=c;

        t1.append(s);

        String q = db.get_qty(item);

        db.update_qty(item,s,q);

        Toast.makeText(getApplicationContext(),"Updated",Toast.LENGTH_SHORT).show();
    }

    public void getTotal(View v) {
        String total = Integer.toString(sum);
        t1.setText("");

        t1.append(total);
    }
}




CONTROLLER

package com.example.manasi.insertintodb3fields;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

/**
 * Created by Manasi on 15-10-2017.
 */

public class DB_Controller extends SQLiteOpenHelper {

    public DB_Controller(Context c) {
        super(c,"tablebla.db",null,1);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL("CREATE TABLE FOOD(ITEM TEXT, COST TEXT, QTY TEXT)");
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldv, int newv) {
        db.execSQL("DROP TABLE IF EXISTS FOOD");
        onCreate(db);
    }

    public void insertitem(String item, String cost, String qty) {
        SQLiteDatabase db = this.getWritableDatabase();
        ContentValues cv = new ContentValues();
        cv.put("ITEM",item);
        cv.put("COST",cost);
        cv.put("QTY",qty);
        db.insert("FOOD",null,cv);
    }

    public Cursor listitem() {
        SQLiteDatabase db = this.getReadableDatabase();
        Cursor c = db.rawQuery("SELECT * FROM FOOD",null);
        return c;
    }

    public String get_cost(String item) {
        String cost="";
        SQLiteDatabase db = this.getReadableDatabase();
        Cursor c = db.query("FOOD",null,"ITEM=?",new String[]{item}, null,null,null);
        if(c.getCount()<1) {
            c.close();
            return "NOT EXIST";
        }
        c.moveToFirst();
        cost = c.getString(c.getColumnIndex("COST"));
        return cost;
    }

    public String get_qty(String item) {
        String qty="";
        SQLiteDatabase db = this.getReadableDatabase();
        Cursor c = db.query("FOOD",null,"ITEM=?",new String[]{item}, null,null,null);
        if(c.getCount()<1) {
            c.close();
            return "NOT EXIST";
        }
        c.moveToFirst();
        qty = c.getString(c.getColumnIndex("QTY"));
        return qty;
    }

   public void update_qty(String item, String cost, String qty) {
       SQLiteDatabase db = this.getWritableDatabase();
       ContentValues cv = new ContentValues();
       int q = Integer.valueOf(qty).intValue();
       if(q==0)
           return;

       else {
           q = q-1;
           qty = Integer.toString(q);
           cv.put("ITEM", item);
           cv.put("COST",cost);
           cv.put("QTY",qty);

           db.update("FOOD",cv,"ITEM=?",new String[]{item});
       }
   }


}
