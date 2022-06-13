## 1. Learning outcome
You can explain what a cloud platform provider is and can deploy (parts of) your application to a cloud platform. You integrate cloud services (for example: Serverless computing, cloud storage, container management) into your enterprise application, and can explain the added value of these cloud services for your application.

## 2. Algolia 
Algolia is a hosted search engine, offering full-text, numerical, and faceted search, capable of delivering real-time results from the first keystroke. Algolia's powerful API lets you quickly and seamlessly implement search within your websites and mobile applications. Our search API powers billions of queries for thousands of companies every month, delivering relevant results in under 100ms anywhere in the world.

### 2.1 Setup
To make use of Angolia some setup is required. Besides creating an account installing the dependencies the existing data needs to be imported. Using the `php artisan scout:import` command the existing data can be imported into the angolia index.
![Algolia import](https://user-images.githubusercontent.com/46562627/173346236-d1e881ee-92a8-49c9-bd4a-63b28b52c471.PNG)


### 2.2 Implementation
```php
class Song extends Model
{
    use Searchable;
    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'id',
        'name',
        'album_id',
        'genre',
        'resourceLocation',
        'releaseDate',
    ];

    protected $table = "song";

    /**
     * The attributes that should be hidden for serialization.
     *
     * @var array<int, string>
     */
    protected $hidden = [

    ];

    /**
     * The attributes that should be cast.
     *
     * @var array<string, string>
     */
    protected $casts = [

    ];

    public function albums()
    {
        return $this->belongsTo(Album::class);
    }

    public function ratings()
    {
        return $this->hasMany(Rating::class);
    }
}
```

```php
public function searchSong(Request $request)
{
    $validateData = $request->validate([
        'query' => 'required|string',
    ]);
    $collection = Song::search($validateData['query'])->get();
    $response = new ValidResponse($collection);
    return response()->json($response, 200);
}
```

### 2.3 Test
Creating a route makes it possible to access the endpoint and test the functionality:

![Algolia search](https://user-images.githubusercontent.com/46562627/173346568-5e4a1f00-eac8-4fc2-9581-790c2c43d151.PNG)

Through the analytics page of of Angolia I can check that the request came through:

![Algolia search monitor](https://user-images.githubusercontent.com/46562627/173346753-bda1da85-ece7-43a3-a7c8-461c858acb0a.PNG)

### 2.3 Scaling
Being aware of the pricing of a chosen cloud service is really important. Algolia provides 3 types of plans; Free, Standard or Premium. The difference between Standard or Premium are mostly down to personalisation (personalise search results per user using AI), dynamic re-ranking (boost well performing results), and Relevant sorts (AI sorted results without data duplication). Basically most of the AI functionalities.
The exact price is based on the amount of records and requests. The more records and requests are made, the cheaper they become per singular unit. 

![Algolia Pricing](https://user-images.githubusercontent.com/46562627/173388317-052067ff-f1b2-47cc-a382-6d35f9856c57.PNG)

Because pricing is based on records and searches, the ammount of records dictates a large pertion of the total price that wil only go up with more data. Therefore it is important to think about where the weakest point is in your own search algorithm and only use a service like Algolia there. For smaller data sets or situations where slower responses are not an issue, Algolia will become just another billing item without any desired results.


