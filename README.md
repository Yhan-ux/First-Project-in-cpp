# First-Project-in-cpp
new repo
Music Playlist Manager 
A console-based Music Playlist Manager that demonstrates an understanding of data structures and algorithms. This system allows users to create playlists, add and remove songs, and generate playlists based on user preferences.
The system uses at least three data structures:
1.	Linked List (for managing a collection of songs in a playlist).
2.	Stack (for undo functionality, to remove the last action taken).
3.	Priority Queue (for sorting songs based on user ratings).
It also implements at least two algorithms:
1.	Sorting Algorithm: To sort songs based on their ratings.
2.	Searching Algorithm: To find a song by its title
Key Concepts Demonstrated:
1.	Linked List: Used for managing a playlist of songs. The Playlist class uses std::list to store songs, supporting efficient addition, removal, and traversal.
2.	Stack: Used for undo functionality, allowing users to undo the last action (add or remove song). The UndoStack class utilizes std::stack to store actions.
3.	Priority Queue: Used to manage songs based on their ratings. The SongPriorityQueue class utilizes std::priority_queue and a custom comparator to ensure that songs are sorted by their ratings.
4.	Sorting Algorithm: The sortByRating()
Song Structure
The Song struct represents an individual song. It stores the title, artist, and rating of the song.
struct Song {
    std::string title;
    std::string artist;
    int rating;  // 1 to 5 stars

    Song(std::string t, std::string a, int r) : title(t), artist(a), rating(r) {}

    // for saving to file, convert song to a string format
    std::string toString() const {
        return title + "," + artist + "," + std::to_string(rating);
    }

    // for loading from file,  string back into Song
    static Song fromString(const std::string& str) {
        std::stringstream ss(str); // create a stringstream object to parse the input string
        std::string title, artist;
        int rating;
        std::getline(ss, title, ','); // read the 'title' from the stringstream until a comma (',') is encountered
        std::getline(ss, artist, ',');  // read the 'artist' from the stringstream until a comma (',') is encountered
        ss >> rating;
        return Song(title, artist, rating); // return the Song object 'title', 'artist', and 'rating'
    }
};
•	title: The title of the song.
•	artist: The artist who performed the song.
•	rating: A rating between 1 and 5 to represent how good the song is.
The constructor initializes these attributes when creating a new song.
CompareSongRating Structure (Comparator)
A comparator CompareSongRating is used for sorting songs in a priority queue by their rating.
// Comparator for priority queue to sort by rating
struct CompareSongRating {
    bool operator()(const Song& a, const Song& b) {
        return a.rating < b.rating;  // Higher ratings have higher priority
    }
};
•	This is a functor (a class with operator() defined) that will be used to compare two Song objects.
•	It sorts songs in descending order of their rating, meaning higher-rated songs will have higher priority in the priority queue.
Playlist Class (Linked List)
This class manages the playlist using a doubly linked list (std::list).

// Linked list for managing songs in a playlist
class Playlist {
private:
    std::list<Song> songs;   // doubly linked list that stores the songs

    // function to convert a string to lowercase
    std::string toLowerCase(const std::string& str) const {
        std::string lowerStr = str;
        std::transform(lowerStr.begin(), lowerStr.end(), lowerStr.begin(), ::tolower);
        return lowerStr;
    }

public:
    // Add song to playlist
    void addSong(const Song& song) {
        songs.push_back(song); // add the song to the end of the list
        std::cout << "Song '" << song.title << "' by " << song.artist << " added to the playlist.\n";
    }

    // Remove song from playlist by title
    void removeSong(const std::string& songTitle) {
        std::string lowerTitle = toLowerCase(songTitle); //convert a string to lowercase
        for (auto it = songs.begin(); it != songs.end(); ++it) {    // iterate through the list of songs
            std::string itlower = toLowerCase(it->title);
            if (itlower== lowerTitle) { 
                std::cout << "Song '" << it->title << "' removed from the playlist.\n";
                songs.erase(it);   // Erase the song from the list
                return;   // Exit the function once the song is removed
            }
        }
        //if not found to the list
        std::cout << "Song '" << songTitle<< "' not found in the playlist.\n";
    }

    // Display all songs in playlist
    void displayPlaylist() const {
        if (songs.empty()) {  // Check if the playlist is empty
            std::cout << "The playlist is empty.\n";
            return;
        }
        std::cout << "********** PLAYLIST **********\n";
        std::cout << std::setw(35) << std::left << "TITLE"
            << std::setw(20) << std::left << "ARTIST"
            << std::setw(10) << std::right << "RATING\n";
        std::cout << std::setfill('-') << std::setw(65) << "" << std::setfill(' ') << '\n';  // for a separator line
        // Iterate over the songs and display their details
        for (const auto& s : songs)  // Iterate over songs
            std::cout << std::setw(35) << std::left << s.title
            << std::setw(20) << std::left << s.artist
            << std::setw(5) << std::right << "[ " << s.rating << "/5 ] \n";
    }

    // Search for a song by title
    bool searchSong(const std::string& title) const {
        bool found = false;
        std::string lowerTitle = toLowerCase(title);
        // convert input to lowercase
     
        for (const auto& song : songs) { // Iterate over songs
            std::string lowerSongTitle = toLowerCase(song.title);
            if (lowerSongTitle.find(lowerTitle) != std::string::npos) { // Check if the song title contains the search string
                std::cout << "Song found: " << song.title << " by " << song.artist << "\n";
                found = true;
            }
        }
        if (!found)  std::cout << "No song found with the title containing: " << title << "\n";
        return found;
    }

    // sort the songs by rating (high to low)
    void sortByRating() {
        // usse the list sort function with a custom comparison (rating in descending order)
        songs.sort([](const Song& a, const Song& b) { return a.rating > b.rating; });
        std::cout << "Playlist sorted by rating (high to low).\n";
    }

    // Generate playlist based on rating preference
    void generateTopPlaylist(int ratingThreshold) {
        std::cout << "Songs with rating >= " << ratingThreshold << ":\n";
        std::cout << std::setw(35) << std::left << "TITLE"
            << std::setw(20) << std::left << "ARTIST"
            << std::setw(10) << std::right << "RATING\n";
        std::cout << std::setfill('-') << std::setw(65) << "" << std::setfill(' ') << '\n';  // for a separator line
       
        // Iterate through all songs and display those with a rating >= ratingThreshold
        for (const auto& s : songs) {
            if (s.rating >= ratingThreshold)
                std::cout << std::setw(35) << std::left << s.title
                << std::setw(20) << std::left << s.artist
                << std::setw(5) << std::right << "[ " << s.rating << "/5 ] \n";
        } 
       
            
    }

    // find song by title (for undo action)
    Song findSong(const std::string& title) const {
        for (const auto& song : songs) {     // Iterate through the songs to find one with the matching title
            if (toLowerCase(song.title) == toLowerCase(title))return song; // Return the song if found
        }
        return Song("", "", 0);  // Return empty song if not found
    }

    // Save the playlist to a file
    void saveToFile(const std::string& filename) const {
        std::ofstream outFile(filename); //ofstream	Creates and writes to files
        if (!outFile) {                                             // from cppreferencce
            std::cerr << "Error opening file for writing.\n";       //cerr means "character error (stream)"
            return;                                                //return if no found file
        }
        // write each song details to the file (using the Song class's toString method)
        for (const auto& song : songs) {
            outFile << song.toString() << "\n";
        }
        outFile.close(); //close the file
        std::cout << "Playlist saved to " << filename << "\n"; // saved the song to the file
    }

    // Load the playlist from a file
    void loadFromFile(const std::string& filename) {
        std::ifstream inFile(filename); //ifstream	Reads from files
        if (!inFile) {
            std::cerr << "Error opening file for reading.\n";        //cerr means "character error (stream)"
            return;
        }
        songs.clear();  // Clear the existing playlist
        std::string line;
        while (std::getline(inFile, line)) {
            if (!line.empty()) {   // Ignore empty lines
                Song song = Song::fromString(line);// Convert the line to a Song object
                addSong(song);// Add the song to the playlist
            }
        }
        inFile.close(); //close the file
        std::cout << "Playlist loaded from " << filename << "\n"; //to comfirm the losd
    }
};
•  addSong: Adds a song to the playlist (songs.push_back()).
•  removeSong: Removes a song by title (iterates through the list to find and erase it).
•  displayPlaylist: Displays all the songs in the playlist.
•  searchSong: Searches for a song by its title in the playlist.
•  sortByRating: Sorts the songs in the playlist by rating using the list::sort method (C++ STL).
•  generateTopPlaylist: Generates a list of songs with ratings greater than or equal to the given threshold.
UndoStack Class (Stack)
This class manages the undo functionality using a stack
// UndoStack class managing undo and redo operations
class UndoStack {
private:
    Playlist& playlist;  //refrence to the playlist to modify it during undo/redo
    std::stack<UndoAction> undoStack;// Stack to store actions for undo
    std::stack<UndoAction> redoStack;// Stack to store actions for redo

public:
    // Constructor that takes playlist by reference to modify it
    UndoStack(Playlist& p) : playlist(p) {}

    // Push an action onto the stack (Add or Remove)
    void pushAction(const UndoAction& action) {
        undoStack.push(action); // Add the action to the undo stack
        // Clear the redo stack since new action invalidates redo history
        while (!redoStack.empty()) {  // Clear the redo stack since a new action has been made and invalidates previous redo history
            redoStack.pop();
        }
    }

    // Undo the last action
    void undoAction() {
        if (!undoStack.empty()) { // Check if there are any actions in the redo stack
            UndoAction lastAction = undoStack.top();// Get the last undone action from the redo stack
            undoStack.pop();// Remove the action from the redo stack

            // Undo based on the action type
            if (lastAction.actionType == UndoAction::ActionType::ADD) {
                playlist.removeSong(lastAction.song.title);  // Remove song if it was added
            }
            else if (lastAction.actionType == UndoAction::ActionType::REMOVE) {
                playlist.addSong(lastAction.song);  // Re-add the song if it was removed
                std::cout << "Restored song: " << lastAction.song.title << "\n";
            }

            // Push the undone action onto the redo stack
            redoStack.push(lastAction);// Push the redone action onto the undo stack to allow undoing it again if needed
        }
        else {
            std::cout << "No actions to undo.\n";
        }
    }

    // Redo the last undone action
    void redoAction() {
        if (!redoStack.empty()) {
            UndoAction lastAction = redoStack.top();
            redoStack.pop();

            // Redo based on the action type
            if (lastAction.actionType == UndoAction::ActionType::ADD) {
                playlist.addSong(lastAction.song);  // Re-add song if it was removed
            }
            else if (lastAction.actionType == UndoAction::ActionType::REMOVE) {
                playlist.removeSong(lastAction.song.title);  // Re-remove song if it was added
            }

            // Push the redone action onto the undo stack
            undoStack.push(lastAction);
        }
        else {
            std::cout << "No actions to redo.\n";
        }
    }
};
•	pushAction: Pushes an action (like "Add song" or "Remove song") onto the undo stack.
•	undoAction: Pops the last action from the stack and prints it. This simulates undoing the last action performed.
