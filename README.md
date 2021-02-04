# flask-marshmallow
Flask + Marshmallow

## Marshmallow
Marshmallow is the serialization library used in python to map schema between input/output and DB.

## Config
```
app = Flask(__name__)
ma = Marshmallow(app)
```

## Database configuration
```
basedir = os.path.abspath(os.path.dirname(__file__))
 
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///' + os.path.join(basedir, 'db.sqlite')
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)
```

## Schema Defination

```

class Book(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(255))
    author_id = db.Column(db.Integer, db.ForeignKey("author.id"))
    author = db.relationship("Author", backref="books")


class AuthorSchema(ma.Schema):
    class Meta:
        model = Author

    id = ma.auto_field()
    name = ma.auto_field()
    books = ma.auto_field()

    _links = ma.Hyperlinks({
        'self': ma.URLFor('Author', values=dict(id='<id>')),
        'collection': ma.URLFor('author_list')
    })

class BookSchema(ma.Schema):
    class Meta:
        fields = ('id', 'title', 'author', 'links')

    author = ma.Nested(AuthorSchema)

    links = ma.Hyperlinks({
        'self': ma.URLFor('book_detail', values=dict(id='<id>')),
        'collection': ma.URLFor('book_list')
    })

user_schema = UserSchema()
users_schema = UserSchema(many=True)
```
## Set Endpoint

```
@app.route("/api/users/<id>")
def user_deail(id):
    user = User.query.get(id)
    return user_schema.dump(user)
    
```    

## Dockerfile

```
FROM python:3

COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
EXPOSE 5002
ENTRYPOINT [ "python" ]
CMD ["app.py"]
```

### Docker compose

```
version: "3.8"
services: 
    web:
        build: .
        ports: 
          - 5002:5002
        volumes: 
          - ./app:/app
       
```

### Reference

https://marshmallow-sqlalchemy.readthedocs.io/en/latest/recipes.html#introspecting-generated-fields

https://pypi.org/project/marshmallow-sqlalchemy/

https://livecodestream.dev/post/python-flask-api-starter-kit-and-project-layout/

