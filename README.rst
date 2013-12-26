=================
django-audiofield
=================

Django-audiofield is a Django application which allows audio file upload and conversion to mp3, wav and ogg format.
It also makes it easy to play the audio files into your django application, for this we integrated a HTML5 and Flash audio player 'SoundManager2'

The goal of this project is to quickly manage audio files into your django project and make it easy for admins and users to listen to them.


.. image:: https://github.com/Star2Billing/django-audiofield/raw/master/docs/source/_static/django-admin-audiofield.png

.. image:: https://github.com/Star2Billing/django-audiofield/raw/master/docs/source/_static/django-admin-audiofield-upload.png

More information about Soundmanager2 : http://www.schillmania.com/projects/soundmanager2/


Installation
============

Install Django-Audiofield::

    python setup.py install


Dependencies
------------

Install dependencies on Debian::

    apt-get -y install libsox-fmt-mp3 libsox-fmt-all mpg321 dir2ogg libav-tools


Install dependencies on Redhat/CentOS::

    yum -y install python-setuptools libsox-fmt-mp3 libsox-fmt-all mpg321 dir2ogg


Install avconv on Redhat/CentOS::

    git clone git://git.libav.org/libav.git
    cd libav
    sudo ./configure --disable-yasm
    sudo make
    sudo make install


Settings
========

in your settings.py file::

    # Set Following variable
    #MEDIA_ROOT = ''
    #MEDIA_URL = ''

    In MIDDLEWARE_CLASSES add 'audiofield.middleware.threadlocals.ThreadLocals'

    In INSTALLED_APPS add 'audiofield'

    # Frontend widget values
    CHANNEL_TYPE_VALUE = 0  # 0-Keep original, 1-Mono, 2-Stereo

    FREQ_TYPE_VALUE = 8000  # 0-Keep original, 8000-8000Hz, 16000-16000Hz, 22050-22050Hz,
                         # 44100-44100Hz, 48000-48000Hz, 96000-96000Hz

    CONVERT_TYPE_VALUE = 0 # 0-Keep original, 1-Convert to MP3, 2-Convert to WAV, 3-Convert to OGG


Usage
=====

Add the following lines in your models.py file::

    from django.conf import settings
    from audiofield.fields import AudioField
    import os.path

    # Add the audio field to your model
    audio_file = AudioField(upload_to='your/upload/dir', blank=True,
                            ext_whitelist=(".mp3", ".wav", ".ogg"),
                            help_text=("Allowed type - .mp3, .wav, .ogg"))

    # Add this method to your model
    def audio_file_player(self):
        """audio player tag for admin"""
        if self.audio_file:
            file_url = settings.MEDIA_URL + str(self.audio_file)
            player_string = '<ul class="playlist"><li style="width:250px;">\
            <a href="%s">%s</a></li></ul>' % (file_url, os.path.basename(self.audio_file.name))
            return player_string
    audio_file_player.allow_tags = True
    audio_file_player.short_description = _('Audio file player')


Add the following lines in your admin.py::


    from your_app.models import your_model_name

    # add 'audio_file_player' tag to your admin view
    list_display = (..., 'audio_file_player', ...)
    actions = ['custom_delete_selected']

    def custom_delete_selected(self, request, queryset):
        #custom delete code
        n = queryset.count()
        for i in queryset:
            if i.audio_file:
                if os.path.exists(i.audio_file.path):
                    os.remove(i.audio_file.path)
            i.delete()
        self.message_user(request, _("Successfully deleted %d audio files.") % n)
    custom_delete_selected.short_description = "Delete selected items"

    def get_actions(self, request):
        actions = super(AudioFileAdmin, self).get_actions(request)
        del actions['delete_selected']
        return actions


If you are not using the installation script, please copy following template
file to your template directory::

    cp audiofield/templates/common_audiofield.html /path/to/your/templates/directory/


Add the following in your template files (like admin/change_form.html, admin/change_list.html etc.
in which you are using audio field type)::


    {% block extrahead %}
    {{ block.super }}
        {% include "common_audiofield.html" %}
    {% endblock %}


Then perform following commands to create the table and collect the static files::

    ./manage.py syncdb
    ./manage.py collectstatic


Create audiofield.log file::

    touch /var/log/audio-field.log


Contributing
============

If you've found a bug, implemented a feature or customized the template and
think it is useful then please consider contributing. Patches, pull requests or
just suggestions are welcome!

Source code: http://github.com/Star2Billing/django-audiofield

Bug tracker: https://github.com/Star2Billing/django-audiofield/issues


Documentation
=============

Documentation is available on 'Read the Docs':
http://django-audiofield.readthedocs.org


Credit
======

Django-audiofield is a Star2Billing-Sponsored Community Project, for more information visit
http://www.star2billing.com or email us at info@star2billing.com


License
=======

django-audiofield is licensed under MIT, see `MIT-LICENSE.txt`.
