import React, { useState, useEffect } from 'react';
import { View, TextInput, StyleSheet, Alert, Image, TouchableOpacity, Text, Dimensions, ActivityIndicator } from 'react-native';
import * as ImagePicker from 'expo-image-picker';
import AsyncStorage from '@react-native-async-storage/async-storage';
import UserPermissions from './utilities/UserPermissions';
import medicozinConfig from '../medicozin.config';
import { Video } from 'expo-av';

const { width } = Dimensions.get('window');

const PostForm = () => {
    const [content, setContent] = useState('');
    const [type, setType] = useState('');
    const [imageUri, setImageUri] = useState(null);
    const [userId, setUserId] = useState(null);
    const [loading, setLoading] = useState(false);
    const maxWords = 3000;

    useEffect(() => {
        const fetchUserId = async () => {
            try {
                const storedId = await AsyncStorage.getItem('id');
                if (storedId !== null) {
                    setUserId(JSON.parse(storedId));
                }
            } catch (error) {
                console.log('Error fetching user ID:', error);
            }
        };

        fetchUserId();
    }, []);

    const handleImagePick = async (type) => {
        await UserPermissions.getMediaLibraryPermissions();

        try {
            let result;
            if (type === 'video') {
                result = await ImagePicker.launchImageLibraryAsync({
                    mediaTypes: ImagePicker.MediaTypeOptions.Videos,
                    allowsEditing: true,
                    quality: 1,
                });
            } else {
                result = await ImagePicker.launchImageLibraryAsync({
                    mediaTypes: ImagePicker.MediaTypeOptions.Images,
                    allowsEditing: true,
                    quality: 1,
                });
            }

            if (!result.canceled && result.assets.length > 0) {
                const uri = result.assets[0].uri;
                if (uri) {
                    setImageUri(uri);
                } else {
                    console.log('Image or video URI is null');
                }
            } else {
                console.log('Image or video selection canceled');
            }
        } catch (error) {
            console.error('Error picking image or video:', error);
        }
    };

    const handleSubmit = async () => {
        if (!content || !type) {
            Alert.alert('Error', 'Content and Type are required');
            return;
        }

        const formData = new FormData();
        formData.append('studentId', userId || ''); // Ensure userId is not null
        formData.append('content', content);
        formData.append('type', type);

        if (imageUri) {
            try {
                const uriParts = imageUri.split('.');
                const fileType = uriParts[uriParts.length - 1];
                formData.append('image', {
                    uri: imageUri,
                    name: `photo.${fileType}`,
                    type: `image/${fileType}`,
                });
            } catch (error) {
                console.error('Error processing image URI:', error);
            }
        }

        setLoading(true);
        try {
            const jwt = await AsyncStorage.getItem('token');

            const response = await fetch(`${medicozinConfig.API_HOST}/create`, {
                method: 'POST',
                body: formData,
                headers: {
                    Accept: 'application/json',
                    'Content-Type': 'multipart/form-data',
                    Authorization: `Bearer ${jwt}`,
                },
            });

            const result = await response.json();

            if (response.ok) {
                Alert.alert('Success', 'Post created successfully');
                setContent('');
                setType('');
                setImageUri(null);
            } else {
                Alert.alert('Error', result.message || 'Something went wrong');
            }
        } catch (error) {
            Alert.alert('Error', 'Network error');
        } finally {
            setLoading(false);
        }
    };

    const handleContentChange = (text) => {
        const words = text.trim().split(/\s+/);
        if (words.length <= maxWords) {
            setContent(text);
        } else {
            Alert.alert('Word Limit Exceeded', `You can only write up to ${maxWords} words.`);
        }
    };

    return (
        <View style={styles.container}>
            <Text style={styles.header}>Create a Post</Text>
            <View style={styles.textinput}>
                <TextInput
                    style={styles.input}
                    placeholder="Enter Post Content..."
                    placeholderTextColor={'#d3d3d3'}
                    value={content}
                    onChangeText={handleContentChange}
                    multiline
                />
                <Text style={styles.wordCount}>{content.trim().split(/\s+/).length}/{maxWords} words</Text>
            </View>
            {!imageUri && type === '' && (
                <View style={styles.iconContainer}>
                    <TouchableOpacity onPress={() => { setType('image'); handleImagePick('image'); }} style={styles.iconButton}>
                        <Image source={require('../assets/picture.png')} style={styles.icon} />
                        <Text style={styles.iconText}>Image</Text>
                    </TouchableOpacity>
                    <TouchableOpacity onPress={() => { setType('video'); handleImagePick('video'); }} style={styles.iconButton}>
                        <Image source={require('../assets/play-button2.png')} style={styles.icon} />
                        <Text style={styles.iconText}>Video</Text>
                    </TouchableOpacity>
                </View>
            )}
            {imageUri && type === 'video' ? (
                <Video
                    source={{ uri: imageUri }}
                    style={styles.image}
                    useNativeControls
                    resizeMode="contain"
                />
            ) : (
                imageUri && (
                    <Image
                        source={{ uri: imageUri }}
                        style={styles.image}
                    />
                )
            )}
            {loading ? (
                <ActivityIndicator size="large" color="#4CAF50" style={styles.loadingIndicator} />
            ) : (
                <TouchableOpacity style={styles.submitButton} onPress={handleSubmit}>
                    <Text style={styles.submitButtonText}>Create Post</Text>
                </TouchableOpacity>
            )}
        </View>
    );
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        padding: 20,
        backgroundColor: '#f5f5dc',
    },
    header: {
        fontSize: 24,
        fontWeight: 'bold',
        color: 'black',
        marginBottom: 16,
        textAlign: 'center',
    },
    textinput: {
        marginBottom: 26,
        backgroundColor:'#414a4c',
        borderRadius:5
    },
    input: {
        height: 490,
        borderColor: 'white',
        borderRadius: 1,
        paddingHorizontal: 12,
        paddingVertical: 8,
        color: '#d3d3d3',
        borderWidth: 0.1,
        textAlignVertical: 'top',
    },
    wordCount: {
        color: '#d3d3d3',
        textAlign: 'right',
        marginTop: 4,
        fontSize: 14,
    },
    image: {
        width: width - 40,
        height: 200,
        borderRadius: 8,
        marginBottom: 16,
        resizeMode: 'cover',
    },
    submitButton: {
        backgroundColor: '#4CAF50',
        padding: 14,
        borderRadius: 8,
        alignItems: 'center',
    },
    submitButtonText: {
        color: '#fff',
        fontSize: 16,
        fontWeight: 'bold',
    },
    iconContainer: {
        flexDirection: 'row',
        justifyContent: 'space-around',
        marginBottom: 180,
    },
    iconButton: {
        alignItems: 'center',
    },
    icon: {
        width: 30,
        height: 30,
        marginBottom: 8,
    },
    iconText: {
        color: 'black',
        fontSize: 16,
        fontWeight: 'bold',
    },
    loadingIndicator: {
        marginVertical: 20,
    },
});

export default PostForm;
