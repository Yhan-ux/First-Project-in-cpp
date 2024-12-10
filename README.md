# First-Project-in-cpp
new repo
#include <iostream>
#include <string>
#include <list>
#include <queue>
#include <stack>
#include <algorithm>
#include <vector>
#include <functional>
#include <iomanip>
#include <fstream>
#include <sstream>
#include <chrono>
#include <ctime>
// Song structure
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

// UndoAction class to store undoable actions (add or remove)
struct UndoAction { //-https://www.w3schools.com/cpp/cpp_enum.asp

   enum class ActionType { ADD, REMOVE }; // enum class to define the type of action being tracked: ADD or REMOVE
    ActionType actionType; // Member variable for store the type of action (either ADD or REMOVE)
    Song song;

    UndoAction(ActionType action, const Song& s) : actionType(action), song(s) {} //initializer list to set the actionType and song
};

// Comparator for priority queue to sort by rating
struct CompareSongRating {
    bool operator()(const Song& a, const Song& b) {
        return a.rating < b.rating;  // Higher ratings have higher priority
    }
};

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
// Priority Queue for managing songs sorted by rating
class SongPriorityQueue {
private:
   
            // The priority queue to store songs, sorted by rating.
            // The CompareSongRating class will be used to define the comparison logic.
    std::priority_queue<Song, std::vector<Song>, CompareSongRating> songQueue;
    
public:
    // function to add a song to the priority queue
    void addSong(const Song& song) {
        songQueue.push(song);  // Adds the song to the priority queue
        std::cout << "Song '" << song.title << "' by " << song.artist << " added to the priority queue.\n";
       
    }
    // function to display the top songs in the priority queue
    void displayTopSongs() const {
        if (songQueue.empty()) {  // Check if the priority queue is empty
            std::cout << "No songs in the priority queue.\n";
            return; // exit the function
        }

        std::cout << "********** Top rated songs in priority queue **********\n";
        std::cout << std::setw(35) << std::left << "TITLE"
            << std::setw(20) << std::left << "ARTIST"
            << std::setw(10) << std::right << "RATING\n";
        std::cout << std::setfill('-') << std::setw(65) << "" << std::setfill(' ') << '\n';  // for a separator line
            // temporary copy of the priority queue so we can iterate over it without modifying the original queue
            std::priority_queue<Song, std::vector<Song>, CompareSongRating> tempQueue = songQueue;
            // Iterate through the songs in the temporary priority queue
            while (!tempQueue.empty()) {
                const Song& s= tempQueue.top(); // Get the song with the highest rating (top of the queue)
                    // Print the song title, artist, and rating to the console
                    std::cout << std::setw(35) << std::left << s.title  
                        << std::setw(20) << std::left << s.artist
                        << std::setw(5) << std::right << "[ " << s.rating << "/5 ] \n";
                    tempQueue.pop();// Remove the song from the temporary queue
            }
          
    }
};


// Main application class
class MusicPlaylistManager {
private:
    Playlist playlist;
    UndoStack undoStack;
    SongPriorityQueue priorityQueue;
    std::string getCurrentTime() { //-https://www.w3schools.com/cpp/ref_ctime_asctime.asp
                                   //-https://www.w3schools.com/cpp/cpp_date.asp
        // Get current time from system clock
        auto now = std::chrono::system_clock::now();
        std::time_t time = std::chrono::system_clock::to_time_t(now);

        // Convert to a readable format using std::localtime_r or std::localtime_s for thread safety
        std::tm localTime;
        #ifdef _WIN32
                localtime_s(&localTime, &time);  // Use localtime_s on Windows
        #else
                localtime_r(&time, &localTime);  // Use localtime_r on POSIX systems
        #endif

        // Format the time (e.g., "YYYY-MM-DD HH:MM:SS")
        std::stringstream timeStream;
        timeStream << std::put_time(&localTime, "%Y-%m-%d %H:%M:%S");

        return timeStream.str();
    }
    
public:
    MusicPlaylistManager() : undoStack(playlist) {}
    void print_time() {
        std::cout << std::setfill('*') << std::setw(20) << "" << std::setfill(' ');
        std::cout << getCurrentTime(); // Display the current time
        std::cout << std::setfill('*') << std::setw(21) << "" << std::setfill(' ');
    }
    void run() {
       int choice;
            playlist.loadFromFile("playlist.txt");  // Load playlist on startup
            playlist.loadFromFile("priority.txt");

               
                  do {
                        std::cout << '\n'<<std::setfill('*') << std::setw(60) << "" << std::setfill(' ') << '\n';
                        std::cout << std::setfill(' ') << std::setw(20) << "" << std::setfill(' ') << std::setw(20);
                        std::cout <<"Current Time : " << getCurrentTime() << "\n"; // Display the current time
                        std::cout << "\nMusic Playlist Manager\n";
                        std::cout << "  1. Add song\n";
                        std::cout << "  2. Remove song\n";
                        std::cout << "  3. Display playlist\n";
                        std::cout << "  4. Search song\n";
                        std::cout << "  5. Sort playlist by rating\n";
                        std::cout << "  6. Generate playlist by rating\n";
                        std::cout << "  7. Undo last action\n";
                        std::cout << "  8. Redo last undone action\n";
                        std::cout << "  9. Add song to priority queue\n";
                        std::cout << " 10. Display top songs in priority queue\n";
                        std::cout << "  \nPress '0' for exit\n";
                        std::cout << std::setfill('*') << std::setw(60) << "" << std::setfill(' ') << '\n';
                        std::cout << "Enter your choice: ";
                        std::cin >> choice;
                    
                    switch (choice) {
                    case 1: { // Add song
                        std::string title, artist;
                        int rating;
                        std::cout << "Enter song title: ";
                        std::cin.ignore();  // to clear the newline from input buffer
                        std::getline(std::cin, title);
                        std::cout << "Enter artist: ";
                        std::getline(std::cin, artist);
                        std::cout << "Enter rating (1 to 5): ";
                        std::cin >> rating;
                        Song song(title, artist, rating);
                        priorityQueue.addSong(song);
                        playlist.addSong(song);
                        undoStack.pushAction(UndoAction(UndoAction::ActionType::ADD, song)); // Push ADD action after adding the song
                        playlist.saveToFile("PlayList.txt");
                        print_time();

                        break;
                    }
                    case 2: { //. Remove song
                        std::string title;
                        std::cout << "Enter song title to remove: ";
                        std::cin.ignore();  // clear the newline
                        std::getline(std::cin, title);
                        Song songToRemove = playlist.findSong(title);  // Find the song object to store for undo
                        if (songToRemove.title.empty()) {
                            std::cout << "Song not found in the playlist.\n";
                        }
                        else {
                            playlist.removeSong(title);
                            undoStack.pushAction(UndoAction(UndoAction::ActionType::REMOVE, songToRemove));  // Push REMOVE action after removing the song
                        }
                        print_time();

                        break;
                    }
                    case 3: //Display playlist
                        playlist.displayPlaylist();
                        print_time();

                        break;
                    case 4: {
                        std::string title;
                        std::cout << "Enter song title to search: ";
                        std::cin.ignore();  // clear the newline
                        std::getline(std::cin, title);
                        playlist.searchSong(title);
                        print_time();

                        break;
                    }
                    case 5: {//Search song
                        playlist.sortByRating();
                        playlist.displayPlaylist();
                        print_time();
                      
                        break;
                    }
                    case 6: {
                        int rate;
                        std::cout << "Enter rating from (1-5): ";
                        std::cin >> rate;
                        playlist.generateTopPlaylist(rate);
                        print_time();

                        break;
                    }
                    case 7:
                        undoStack.undoAction();// Call undo action here
                        playlist.displayPlaylist();
                        print_time();

                        break;
                    case 8:
                        undoStack.redoAction();  // Call redo action here
                        // Call undo action here
                       print_time();
                        break;
                    case 9: {
                        std::string title, artist;
                        int rating;
                        std::cout << "Enter song title: ";
                        std::cin.ignore();  // clear the newline
                        std::getline(std::cin, title);
                        std::cout << "Enter artist: ";
                        std::getline(std::cin, artist);
                        std::cout << "Enter rating (1 to 5): ";
                        std::cin >> rating;
                        Song song(title, artist, rating);
                        priorityQueue.addSong(song);                      
                        print_time();
                        
                        break;
                    }
                    case 10:

                        priorityQueue.displayTopSongs();
                        print_time();

                        break;
                    case 0:
                     
                        playlist.saveToFile("playlist.txt");  // Save playlist to file on exit
                        std::cout << "Exiting program.\n";
                        print_time();

                        break;
                    default:
                        std::cout << "Invalid choice. Please try again.\n";
                       
                    }
                    
                  } while (choice != 0 );
          
    }
};

// Main function
int main() {
    std::cout << std::setfill('*') << std::setw(55) << "" << std::setfill(' ') << '\n';
    std::cout << "      -MUSIC PLAYLIST MANAGER-\n";
    std::cout << "     Programmed by: Ryan Morao\n";
    std::cout << std::setfill('*') << std::setw(55) << "" << std::setfill(' ') << '\n';
    MusicPlaylistManager manager;
    manager.run();
    return 0;
}
