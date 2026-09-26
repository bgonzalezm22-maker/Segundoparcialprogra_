import { useFocusEffect } from 'expo-router';
import { useCallback, useState } from 'react';
import {
    ActivityIndicator,
    FlatList,
    StyleSheet,
    Text,
    TextInput,
    TouchableOpacity,
    View,
} from 'react-native';

import { StudentRepository, type Student } from '@/repositories/StudentRepository';

const studentRepo = new StudentRepository();

export default function AlumnosScreen() {
  const [students, setStudents] = useState<Student[]>([]);
  const [loading, setLoading] = useState(true);
  const [searchQuery, setSearchQuery] = useState('');

  const fetchStudents = useCallback(async () => {
    setLoading(true);
    try {
      const data = await studentRepo.getAll();
      setStudents(data);
    } catch (error) {
      const message = error instanceof Error ? error.message : 'Error desconocido';
      console.error('Error al obtener estudiantes:', message);
    } finally {
      setLoading(false);
    }
  }, []);

  useFocusEffect(
    useCallback(() => {
      void fetchStudents();
    }, [fetchStudents])
  );

  // Filtrado de estudiantes según el texto ingresado
  const filteredStudents = students.filter((student) => {
    const query = searchQuery.toLowerCase();
    return (
      student.nombre.toLowerCase().includes(query) ||
      student.carnet.toLowerCase().includes(query) ||
      student.carrera.toLowerCase().includes(query)
    );
  });

  const handleEdit = (student: Student) => {
    // Aquí puedes navegar a la pantalla de edición, ej: router.push(`/edit/${student.id}`)
    console.log('Editar estudiante:', student);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Listado de Estudiantes</Text>

      {/* Input de Búsqueda */}
      <View style={styles.searchContainer}>
        <TextInput
          style={styles.searchInput}
          placeholder="Buscar por nombre, carnet o carrera..."
          placeholderTextColor="#A0AEC0"
          value={searchQuery}
          onChangeText={setSearchQuery}
          clearButtonMode="while-editing"
        />
      </View>

      {loading ? (
        <ActivityIndicator size="large" color="#007AFF" />
      ) : (
        <FlatList
          data={filteredStudents}
          keyExtractor={(item) => item.id.toString()}
          renderItem={({ item }) => (
            <View style={styles.card}>
              <View style={styles.cardHeader}>
                <View style={styles.cardInfo}>
                  <Text style={styles.cardName}>{item.nombre}</Text>
                  <Text style={styles.cardDetail}>Carnet: {item.carnet}</Text>
                  <Text style={styles.cardDetail}>Carrera: {item.carrera}</Text>
                </View>

                {/* Botón de Editar */}
                <TouchableOpacity
                  style={styles.editButton}
                  onPress={() => handleEdit(item)}
                >
                  <Text style={styles.editButtonText}>Editar</Text>
                </TouchableOpacity>
              </View>
            </View>
          )}
          ListEmptyComponent={
            <Text style={styles.empty}>
              {searchQuery ? 'No se encontraron resultados.' : 'No hay estudiantes registrados.'}
            </Text>
          }
        />
      )}

      <TouchableOpacity style={styles.fab} onPress={() => undefined}>
        <Text style={styles.fabText}>+</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#F5F7FA',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 16,
    color: '#1A202C',
  },
  searchContainer: {
    marginBottom: 16,
  },
  searchInput: {
    backgroundColor: '#FFFFFF',
    paddingHorizontal: 16,
    paddingVertical: 12,
    borderRadius: 10,
    fontSize: 15,
    color: '#2D3748',
    borderWidth: 1,
    borderColor: '#E2E8F0',
  },
  card: {
    backgroundColor: '#FFFFFF',
    padding: 16,
    borderRadius: 12,
    marginBottom: 12,
    shadowColor: '#000',
    shadowOpacity: 0.05,
    shadowRadius: 5,
    elevation: 2,
  },
  cardHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
  },
  cardInfo: {
    flex: 1,
    marginRight: 10,
  },
  cardName: {
    fontSize: 18,
    fontWeight: '600',
    color: '#2D3748',
  },
  cardDetail: {
    fontSize: 14,
    color: '#718096',
    marginTop: 4,
  },
  editButton: {
    backgroundColor: '#EBF8FF',
    paddingVertical: 8,
    paddingHorizontal: 14,
    borderRadius: 8,
    borderWidth: 1,
    borderColor: '#BEE3F8',
  },
  editButtonText: {
    color: '#3182CE',
    fontWeight: '600',
    fontSize: 14,
  },
  empty: {
    textAlign: 'center',
    marginTop: 40,
    color: '#A0AEC0',
  },
  fab: {
    position: 'absolute',
    right: 20,
    bottom: 30,
    backgroundColor: '#007AFF',
    width: 60,
    height: 60,
    borderRadius: 30,
    justifyContent: 'center',
    alignItems: 'center',
    elevation: 5,
  },
  fabText: {
    color: '#FFF',
    fontSize: 30,
    lineHeight: 32,
  },
});
