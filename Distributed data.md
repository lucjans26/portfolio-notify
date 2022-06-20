## 1. Learning outcome
You are aware of specific data requirements for enterprise systems. You apply best practices for distributed data during your whole development process, both for non-functional and functional requirements. You especially take legal and ethical issues into consideration.

## 2. GDPR 
It is important to be really careful with personal data. The first step to this is to prevent saving personal data that isn't neccesary in the first place. 

## 3. Event Sourcing
Within music streaming services it is really common that songs or even entire albums have to be changed up multiple times. Sometimes to release diffrent versions but sometimes its as simple as having to change a title or some metadata. Having people do this makes it really hard to audit it something goes wrong. Therefore [event sourcing](https://martinfowler.com/eaaDev/EventSourcing.html) is a good way to create an audit log. It logs everything that happens to a certain record or entry. The result of retrieving a certain record is therefore never based on a single field or record but a "sum" of all the transactions.

### 3.1 Setup
The API needs an event model and a database migration that matches quite closely with the record to be sourced. In the example provided I have applied the theorie to a song entity. 

### 3.2 Implementation
Firstly an event model is created which is almost a carbon copy to the original. The only thing that is really added is an `action type`. This variable declares what action exactly was undertaken to create this result. 
```php
class SongEvent extends Model
{
    protected $fillable = [
        'id',
        'song_id',
        'action_type',
        'name',
        'album_id',
        'genre',
        'resourceLocation',
        'releaseDate',
    ];

    protected $table = "song_events";
}
```
The [migration](https://laravel.com/docs/9.x/migrations) is merely a way to set up the needed database tables.

![image](https://user-images.githubusercontent.com/46562627/174597793-78902a53-e088-4f5a-8211-b7b6eaa4812b.png)

The uploadSong method has some extra logic now. After uploading the song to the file storage, and creating a song record, a SongEvent record is created. This logs the action of uploading. The same is then applied to anything that can happen to the song from this point out. For example updating, deleting, etc.
```php
$path = Storage::disk('azure-file-storage')->put("" ,$request->file('song'));

$song = new Song([
    'name' => $validateData['title'],
    'genre' => $validateData['genre'],
    'album_id' => $validateData['album_id'],
    'resourceLocation' => $path,
    'releaseDate' => now(),
]);

$album->songs()->save($song);
$songEvent = new SongEvent([
    'song_id' => $song->id,
    'action_type' => 'upload',
    'name' => $song->name,
    'album_id' => $song->album_id,
    'genre' => $song->genre,
    'resourceLocation' => $song->resourceLocation,
    'releaseDate' => $song->releaseDate,
]);
$songEvent->save();
```

### 3.3 Test
We can simply test this out by uploading a song. After the song has been uploaded and upload SongEvent record should have been created:

![image](https://user-images.githubusercontent.com/46562627/174596390-e4947b8d-3c75-489a-b9eb-1ed1dd5f8351.png)

And after deleting the song, another event should've been added:

![image](https://user-images.githubusercontent.com/46562627/174597208-abf15f5d-b0b9-428b-9b41-a1a9069fffa7.png)

![image](https://user-images.githubusercontent.com/46562627/174597417-d1d75b70-cb67-4f11-bb78-5ac75b971831.png)

