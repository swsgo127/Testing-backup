package app.com.sss.app1;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;

import org.apache.commons.io.FileUtils;

import java.io.File;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import app.com.sss.app1.model.Repo;
import app.com.sss.app1.network.APIServer;
import io.realm.Realm;
import io.realm.RealmResults;
import io.realm.Sort;
import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;
import retrofit2.Retrofit;
import retrofit2.converter.gson.GsonConverterFactory;

public class SimpleToDoActivity extends AppCompatActivity {

    ArrayList<String> items;
//    ArrayAdapter<String>itemsAdapter;
//    ListView lvItems;
    RecyclerView rvItems;
    EditText etNewItem;
    Button btnAddItem;

    LinearLayoutManager layoutManager;
    ToDoRecyclerViewAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_simple_to_do);

        etNewItem = (EditText) findViewById(R.id.etNewItem);
        rvItems = (RecyclerView) findViewById(R.id.rvItems);
        btnAddItem = (Button) findViewById(R.id.btnAddItem);

        setUpBtnAddItem();

//        lvItems = (ListView)findViewById(R.id.lvItems);
//        items = new ArrayList<>();
        readItems();

        layoutManager = new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false);
        adapter = new ToDoRecyclerViewAdapter(items);
        rvItems.setLayoutManager(layoutManager);
        rvItems.setAdapter(adapter);

//        itemsAdapter = new ArrayAdapter<>(this,android.R.layout.simple_list_item_1,items);

//        lvItems.setAdapter(itemsAdapter);
//        items.add("First Item");
//        items.add("Second Item");

//        setupListViewListener();
    }

    private void setUpBtnAddItem() {
        btnAddItem.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
//                onAddItem(v);
                getRepoOfUser();
            }
        });
    }

    public void onAddItem(View v){
        String itemText = etNewItem.getText().toString();
        items.add(itemText);
        adapter.notifyDataSetChanged();
        rvItems.smoothScrollToPosition(items.size() - 1);
//        itemsAdapter.add(itemText);

        etNewItem.setText("test");
        writeItems();
    }

    public void getRepoOfUser() {
        String itemText = etNewItem.getText().toString();
        Retrofit retrofit =
                new Retrofit.Builder()
                        .baseUrl("https://api.github.com/")
                        .addConverterFactory(GsonConverterFactory.create())
                        .build();


        APIServer apiServer = retrofit.create(APIServer.class);
        Call<List<Repo>> call = apiServer.getRepos(itemText);
        call.enqueue(new Callback<List<Repo>>() {
            @Override
            public void onResponse(Call<List<Repo>> call, Response<List<Repo>> response) {
                Log.i("Retrofit", "success");
                final List<Repo> repos = response.body();
                for(Repo repo: repos) {
                    Log.i("Retrofit", "repo id   : " + repo.getId());
                    Log.i("Retrofit", "repo name : " + repo.getName());
                }

                Realm realm = Realm.getDefaultInstance();
                realm.executeTransaction(new Realm.Transaction() {
                    @Override
                    public void execute(Realm realm) {
                        realm.copyToRealmOrUpdate(repos);
                    }
                });
                realm.close();

                readItems();
                adapter.setDataSet(items);
            }

            @Override
            public void onFailure(Call<List<Repo>> call, Throwable t) {
                Log.e("Retrofit", "failure", t);
            }
        });

    }

//    private void setupListViewListener(){
//        lvItems.setOnItemLongClickListener(new AdapterView.OnItemLongClickListener(){
//                    @Override
//                    public boolean onItemLongClick(AdapterView<?> adapter, View item, int pos, long id){
//                        items.remove(pos);
//                        itemsAdapter.notifyDataSetChanged();
//                        writeItems();
//                        return true;
//                    }
//                });
//    }

    private void readItems(){
//        File filesDir = getFilesDir();
//        File todoFile = new File(filesDir, "todo.txt");
//        try{
//            items = new ArrayList<>(FileUtils.readLines(todoFile));
//        }catch (IOException e){
//            items = new ArrayList<>();
//        }
        items = new ArrayList<>();
        Realm realm = Realm.getDefaultInstance();
        RealmResults<Repo> repos = realm.where(Repo.class).findAllSorted("name", Sort.ASCENDING);
        for (Repo repo: repos) {
            items.add(repo.getName());
        }
        realm.close();
    }
    private void writeItems(){
        File filesDir = getFilesDir();
        File todoFile = new File(filesDir, "todo.txt");
        try{
            FileUtils.writeLines(todoFile,items);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
