import datetime
import sys
from flask import Flask, abort
from flask_restful import Api, Resource
from flask_restful import reqparse
from flask_restful import inputs, fields, marshal_with
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy import and_


app = Flask(__name__)
db = SQLAlchemy(app)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///name.db'


# write your code here
class Events(db.Model):
    __tablename__ = 'events'
    id = db.Column(db.Integer, primary_key=True)
    event = db.Column(db.String(80), nullable=False)
    date = db.Column(db.Date, nullable=False)


db.create_all()
api = Api(app)
resource_field = {
        'id': fields.Integer,
        'event': fields.String,
        'date': fields.DateTime(dt_format='iso8601')
    }


class EventPost(Resource):
    @marshal_with(resource_field)
    def get(self):
        parser = reqparse.RequestParser()
        parser.add_argument(
            'start_time',
            type=str,
            default='',
            help="The event date with the correct format is required! The correct format is YYYY-MM-DD!",
        )
        parser.add_argument(
            'end_time',
            type=inputs.date,
            default='',
            help="The event date with the correct format is required! The correct format is YYYY-MM-DD!",
        )
        args = parser.parse_args()
        res = Events.query.filter(and_(Events.date >= args['start_time'], Events.date <= args['end_time'])).all()
        if not res:
            return Events.query.all()
        else:
            return res

    def post(self):
        parser = reqparse.RequestParser()
        parser.add_argument(
            'event',
            type=str,
            help="The event name is required!",
            required=True
        )
        parser.add_argument(
            'date',
            type=inputs.date,
            help="The event date with the correct format is required! The correct format is YYYY-MM-DD!",
            required=True
        )
        args = parser.parse_args()
        ev = Events(event=args['event'], date=args['date'])
        db.session.add(ev)
        db.session.commit()
        result = {
            'message': 'The event has been added!',
            'event': args['event'],
            'date': str(args['date'].date())
        }
        return result


class EventByID(Resource):
    @marshal_with(resource_field)
    def get(self, event_id):
        event = Events.query.filter(Events.id == int(event_id)).first()
        if event is None:
            abort(404, "The event doesn't exist!")
        return event

    def delete(self, event_id):
        event = Events.query.filter(Events.id == event_id).first()
        if event is None:
            abort(404, "The event doesn't exist!")
        else:
            db.session.delete(event)
            db.session.commit()
            return { "message": "The event has been deleted!"}


class EventToday(Resource):
    @marshal_with(resource_field)
    def get(self):
        return Events.query.filter(Events.date == datetime.date.today()).all()


api.add_resource(EventPost, '/event')
api.add_resource(EventToday, '/event/today')
api.add_resource(EventByID, '/event/<int:event_id>')
# do not change the way you run the program
if __name__ == '__main__':
    if len(sys.argv) > 1:
        arg_host, arg_port = sys.argv[1].split(':')
        app.run(host=arg_host, port=arg_port)
    else:
        app.run()
