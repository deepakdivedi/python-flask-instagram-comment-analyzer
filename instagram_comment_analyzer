from flask_restful import Resource, Api
from InstagramAPI import InstagramAPI
from flask import Flask, request
import subprocess as p
import os
import sys
import time
import json
import requests
import argparse
import io
import re
import nltk
from nltk.corpus import stopwords
from nltk.stem.porter import PorterStemmer
import nltk.classify.util
from nltk.classify import NaiveBayesClassifier
from nltk.corpus import names
from nltk.tokenize import word_tokenize

positive_vocab = [ 'awesome', 'outstanding', 'fantastic', 'terrific', 'good', 'nice', 'great', ':)' ]
negative_vocab = [ 'bad', 'terrible','useless', 'hate', ':(' ]
neutral_vocab = [ 'movie','the','sound','was','is','actors','did','know','words','not' ]


app = Flask(__name__)
api = Api(app)


API = InstagramAPI("","")

class login(Resource):
    def put(self):
        global API
        data = request.get_json()
        API = InstagramAPI(data['username'],data['password'])
        API.login()
        #print(API.LastJson)
        if 'invalid_credentials' in API.LastJson :
            return {'login' : False,'message' : API.LastJson['message']}
        return {'login' : True }
class userFeed(Resource):
    def put(self):
        global API
        data = request.get_json()
        max_id = data["max_id"]
        API.getSelfUserFeed(maxid=max_id)
        if API.LastJson['more_available'] is not True:
            hmp = False
        else:
            hmp = True
        max_id = API.LastJson.get('next_max_id','')
        return {"feed": API.LastJson['items'],"hmp":hmp,"max_id":max_id}


class getComments(Resource):
    def get(self):
        global API
        data = request.get_json()
        media_id = data["media_id"]
        comment_count = data["comment_count"]
        API.getMediaComments(media_id,comment_count)
        return API.LastJson

class analyseComments(Resource):
    def put(self):
        global API
        data = request.get_json()
        media_id = data["media_id"]
        comment_count = data["comment_count"]
        API.getMediaComments(str(media_id),str(comment_count))
        data=API.LastJson
        comments=[]
        for comment in data['comments']:
            comments.append(comment['text'])
        #print(comments)
        #print(type(comments))
        s=comments[0]
        for i in range(1,len(comments)):
            s=s+','+comments[i]
        def word_feats(words):
            return dict([(word, True) for word in words])
        corpus = []
        # To clean Comments
        for dataComment in s:

            # column : "comment", row ith
            comment = re.sub('[^a-zA-Z]', ' ', dataComment)

            # convert all cases to lower cases
            comment = comment.lower()

            # split to array(default delimiter is " ")
            comment = comment.split()

            # creating PorterStemmer object to
            # take main stem of each word
            ps = PorterStemmer()

            # loop for stemming each word
            # in string array at ith row
            comment = [ps.stem(word) for word in comment
                        if not word in set(stopwords.words('english'))]

            # rejoin all string array elements
            # to create back into a string
            comment = ' '.join(comment)

            # append each string to create
            # array of clean text
            corpus.append(comment)

        positive_features = [(word_feats(pos), 'pos') for pos in positive_vocab]
        negative_features = [(word_feats(neg), 'neg') for neg in negative_vocab]
        neutral_features = [(word_feats(neu), 'neu') for neu in neutral_vocab]

        train_set = negative_features + positive_features + neutral_features

        classifier = NaiveBayesClassifier.train(train_set)

        neg = 0
        pos = 0
        neu = 0

        words = word_tokenize(str(corpus))
        for word in words:
            classResult = classifier.classify( word_feats(word))
            if classResult == 'neg':
                neg = neg + 1
            if classResult == 'pos':
                pos = pos + 1
            if classResult == 'neu':
                neu = neu + 1

        pos=float(pos)/len(words)
        neg=float(neg)/len(words)
        neu=float(neu)/len(words)
        analysis = 'Positive : ' + str(pos) + ' Negative : ' + str(neg) + ' Neutral : ' + str(neu)
        return {'Analysis':analysis}

class userInfo(Resource):
    def get(Self):
        global Api
        API.getProfileData()
        js1 = API.LastJson
        user_Id = js1['user']['pk']
        API.getUsernameInfo(user_Id)
        js2 = API.LastJson
        return {'userData':js1['user'],'userType':js2['user']['category']}

class logOut(Resource):
    def get(Self):
        global API
        API.logout()
        return {"logout":True}

api.add_resource(analyseComments,"/analysecomments")
api.add_resource(getComments,"/getcomments")
api.add_resource(login,"/login")
api.add_resource(userFeed,"/userfeed")
api.add_resource(logOut,"/logout")
api.add_resource(userInfo,"/userinfo")

if __name__ == '__main__':
     app.run(host='127.0.0.1',port=2000)
